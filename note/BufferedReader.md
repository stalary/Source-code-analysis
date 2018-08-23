- [介绍](#介绍)
- [const&field](#constfield)
- [constructor](#constructor)
- [ensureOpen](#ensureOpen)
- [mark&reset](#markreset)
- [fill](#fill)
- [read](#read)
- [readline](#readline)


### 介绍

BufferedReader继承自Reader，提供高效字符读取。缓冲区的大小可以指定，否则使用默认大小。大多数情况下默认大小就够用的。
每次Reader的读取请求都会产生相应的对字符或字节流的读取请求，所以最好用BufferedReader包装那些read操作影响效率的Reader，比如FileReader和InputStreamReader。

### const&field

```java

// 内部包装的Reader
private Reader in;  


// 缓冲区  
private char cb[];  

//nChars:缓冲区字符总数;nextChar:要读的下一个字符的位置
private int nChars, nextChar;  

// 标记无效
private static final int INVALIDATED = -2;  

// 未设置标记 
private static final int UNMARKED = -1;  

// 标记
private int markedChar = UNMARKED;  

// 标记有效的最大长度
private int readAheadLimit = 0; 

// 是否忽略换行符
private boolean skipLF = false;  

// 标记时，保存的skipLF值  
private boolean markedSkipLF = false;  

// 缓冲区默认大小 2^13
private static int defaultCharBufferSize = 8192;  

// 每一行默认字符个数
private static int defaultExpectedLineLength = 80; 


```


### constructor

```java

//不指定缓冲区大小就用默认大小
public BufferedReader(Reader in) {
    this(in, defaultCharBufferSize);
}

//按照给定的缓冲区大小构造
public BufferedReader(Reader in, int sz) {
    super(in);
    if (sz <= 0)
        throw new IllegalArgumentException("Buffer size <= 0");
    this.in = in;
    cb = new char[sz];
    nextChar = nChars = 0;
}

```

### ensureOpen

```java

//在执行对流的操作时，先确保流未被关闭
private void ensureOpen() throws IOException {
    if (in == null)
        throw new IOException("Stream closed");
}

```

### mark&reset

```java

//***下面都是同步的***

//标记，传入标记有效的范围，意味着至少缓冲区大小不能小于这个limit
public void mark(int readAheadLimit) throws IOException {
    if (readAheadLimit < 0) {
        throw new IllegalArgumentException("Read-ahead limit < 0");
    }
    synchronized (lock) {
        ensureOpen();
        this.readAheadLimit = readAheadLimit;
        //标记下一个读取位置
        markedChar = nextChar;
        //保存skipLF的值
        markedSkipLF = skipLF;
    }
}

//重置回标记位置
public void reset() throws IOException {
    synchronized (lock) {
        ensureOpen();
        if (markedChar < 0)
            throw new IOException((markedChar == INVALIDATED)
                                  ? "Mark invalid"
                                  : "Stream not marked");
        //重置
        nextChar = markedChar;
        skipLF = markedSkipLF;
    }
}


```

### fill 

```java

//填充缓冲区，如果标记有效就要考虑在内
private void fill() throws IOException {
    //新读入的存放的位置
    int dst;
    //无标记或标记无效
    if (markedChar <= UNMARKED) {
        dst = 0;
    } else {
        //标记有效
        int delta = nextChar - markedChar;
        if (delta >= readAheadLimit) {
            //超出了标记有效的范围，设为无效
            markedChar = INVALIDATED;
            readAheadLimit = 0;
            dst = 0;
        } else {
            if (readAheadLimit <= cb.length) {
                //将标记的位置拷贝到头上
                System.arraycopy(cb, markedChar, cb, 0, delta);
                markedChar = 0;
                dst = delta;
            } else {
                /* Reallocate buffer to accommodate read-ahead limit */
                //缓冲区容量比readaheadlimit小，就开一块新空间
                char ncb[] = new char[readAheadLimit];
                System.arraycopy(cb, markedChar, ncb, 0, delta);
                cb = ncb;
                markedChar = 0;
                dst = delta;
            }
            //fill方法只会在nextChar>=nChars时才会调用，所以此时缓冲区内字符至少读过一次了
            nextChar = nChars = delta;
        }
    }

    int n;
    do {
        //从Reader中读一批字符过来，放满缓冲区
        n = in.read(cb, dst, cb.length - dst);
    } while (n == 0);
    if (n > 0) {
        nChars = dst + n;
        nextChar = dst;
    }
}

```

### read

```java

//读一个
public int read() throws IOException {
    synchronized (lock) {
        ensureOpen();
        for (;;) {
            if (nextChar >= nChars) {
                fill();
                //这时候输入流读光了
                if (nextChar >= nChars)
                    return -1;
            }
            if (skipLF) {
                skipLF = false;
                //跳过换行
                if (cb[nextChar] == '\n') {
                    nextChar++;
                    continue;
                }
            }
            return cb[nextChar++];
        }
    }
}

//读到cbuf中
public int read(char cbuf[], int off, int len) throws IOException {
    synchronized (lock) {
        ensureOpen();
        if ((off < 0) || (off > cbuf.length) || (len < 0) ||
            ((off + len) > cbuf.length) || ((off + len) < 0)) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return 0;
        }

        int n = read1(cbuf, off, len);
        if (n <= 0) return n;
        //没有读够给定长度，且in可读，读完剩下的(之前可能阻塞了)
        while ((n < len) && in.ready()) {
            int n1 = read1(cbuf, off + n, len - n);
            if (n1 <= 0) break;
            n += n1;
        }
        return n;
    }
}

```

### readline

```java

//读取一行文本
String readLine(boolean ignoreLF) throws IOException {
    StringBuffer s = null;
    int startChar;

    synchronized (lock) {
        ensureOpen();
        boolean omitLF = ignoreLF || skipLF;

    bufferLoop:
        for (;;) {

            if (nextChar >= nChars)
                fill();
            //到达EOF
            if (nextChar >= nChars) {
                if (s != null && s.length() > 0)
                    return s.toString();
                else
                    return null;
            }
            boolean eol = false;
            char c = 0;
            int i;

            //跳过\n
            if (omitLF && (cb[nextChar] == '\n'))
                nextChar++;
            skipLF = false;
            omitLF = false;

        charLoop:
            for (i = nextChar; i < nChars; i++) {
                c = cb[i];
                //找到换行符的位置
                if ((c == '\n') || (c == '\r')) {
                    eol = true;
                    break charLoop;
                }
            }

            startChar = nextChar;
            nextChar = i;

            if (eol) {
                String str;
                 //读取到的一行
                if (s == null) {
                    str = new String(cb, startChar, i - startChar);
                } else {
                    s.append(cb, startChar, i - startChar);
                    str = s.toString();
                }
                nextChar++;
                //标记要下次跳过\r后的\n
                if (c == '\r') {
                    skipLF = true;
                }
                return str;
            }

            if (s == null)
                s = new StringBuffer(defaultExpectedLineLength);
            s.append(cb, startChar, i - startChar);
        }
    }
}


```