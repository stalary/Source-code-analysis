### 介绍

BufferedReader继承自Reader，提供高效字符读取。缓冲区的大小可以指定，否则使用默认大小。大多数情况下默认大小就够用的。
每次Reader的读取请求都会产生相应的对字符或字节流的读取请求，所以最好用BufferedReader包装那些read操作影响效率的Reader，比如FileReader和InputStreamReader。

### const&field

```java

// 内部包装的Reader
private Reader in;  


// 缓冲区  
private char cb[];  

//nChars:缓冲区字符总数; nextChar:要读的下一个字符的位置 private int nChars, nextChar;  

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