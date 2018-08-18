- [介绍](#%E4%BB%8B%E7%BB%8D)
- [构造函数](#%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)
- [getFD](#getfd)
- [getChannel](#getchannel)
- [read](#read)
- [skipBytes](#skipbytes)
- [write](#write)
- [close](#close)
### 介绍
1. 同时支持读和写操作
2. 任意访问文件,可以指定任何一个位置操作文件

### 构造函数
```java
    public RandomAccessFile(File file, String mode)
        throws FileNotFoundException
    {
        // 当file不为空时，获取文件的路径
        String name = (file != null ? file.getPath() : null);
        int imode = -1;
        // 只读方式打开
        if (mode.equals("r"))
            imode = O_RDONLY;
        // 读写模式，文件不存在，则尝试创建该文件
        else if (mode.startsWith("rw")) {
            imode = O_RDWR;
            rw = true;
            if (mode.length() > 2) {
                // 对文件的内容或元数据的每个更新都同步写入到底层存储设备。
                if (mode.equals("rws"))
                    imode |= O_SYNC;
                // 对文件内容的每个更新都同步写入到底层存储设备
                else if (mode.equals("rwd"))
                    imode |= O_DSYNC;
                else
                    imode = -1;
            }
        }
        // 模式无法匹配时，抛出异常
        if (imode < 0)
            throw new IllegalArgumentException("Illegal mode \"" + mode
                                               + "\" must be one of "
                                               + "\"r\", \"rw\", \"rws\","
                                               + " or \"rwd\"");
        // 安全管理器，检测操作是否合法
        SecurityManager security = System.getSecurityManager();
        if (security != null) {
            // 读检测
            security.checkRead(name);
            if (rw) {
                // 写检测
                security.checkWrite(name);
            }
        }
        // 未获取到文件路径，直接抛出空指针异常
        if (name == null) {
            throw new NullPointerException();
        }
        // 文件非法，抛出非法路径异常
        if (file.isInvalid()) {
            throw new FileNotFoundException("Invalid file path");
        }
        // 传入信息
        fd = new FileDescriptor();
        fd.attach(this);
        path = name;
        // 打开文件
        open(name, imode);
    }
```

### getFD
```java
    // 获取文件修饰符
    public final FileDescriptor getFD() throws IOException {
        if (fd != null) {
            return fd;
        }
        throw new IOException();
    }
```

### getChannel
```java
    // 获取channel
    public final FileChannel getChannel() {
        // 加锁
        synchronized (this) {
            // channel为null时打开一个读写模式的channel
            if (channel == null) {
                channel = FileChannelImpl.open(fd, path, true, rw, this);
            }
            return channel;
        }
    }
```

### read
```java
    // 读取字节数组
    public int read(byte b[], int off, int len) throws IOException {
        // 调用native方法进行读取
        return readBytes(b, off, len);
    }
```

```java
    // 从开始位置读取所有字节
    public final void readFully(byte b[]) throws IOException {
        readFully(b, 0, b.length);
    }
```

```java
    // 读取int
    public final int readInt() throws IOException {
        // 读取四个字节
        int ch1 = this.read();
        int ch2 = this.read();
        int ch3 = this.read();
        int ch4 = this.read();
        // 边界检测
        if ((ch1 | ch2 | ch3 | ch4) < 0)
            throw new EOFException();
        // 按位依次返回
        return ((ch1 << 24) + (ch2 << 16) + (ch3 << 8) + (ch4 << 0));
    }
```

```java
    // 读取文本的下一行
    public final String readLine() throws IOException {
        StringBuffer input = new StringBuffer();
        int c = -1;
        // 判断是否到达行尾
        boolean eol = false;

        // 在windows中：\r回车回到行首，不会换到下一行，\n换行到当前位置的下一行，不会回到行首，每行结尾是\r\n而mac是\r
        while (!eol) {
            switch (c = read()) {
            case -1:
            case '\n':
                // 读取换行符，停止
                eol = true;
                break;
            case '\r':
                // 读取回车符，停止
                eol = true;
                long cur = getFilePointer();
                // 对windows系统特殊判断，下一个字符不是\n时，代表是下一行
                if ((read()) != '\n') {
                    seek(cur);
                }
                break;
            default:
                // 将读入的字符添加到字符串中
                input.append((char)c);
                break;
            }
        }
        // 当未读取到字符串，返回null
        if ((c == -1) && (input.length() == 0)) {
            return null;
        }
        // 返回字符串
        return input.toString();
    }
```

```java
    // 读取一个字符串
    public final String readUTF() throws IOException {
        return DataInputStream.readUTF(this);
    }
```

### skipBytes
```java
    // 跳过指定字节数
    public int skipBytes(int n) throws IOException {
        long pos;
        long len;
        long newpos;
        // 非法检测
        if (n <= 0) {
            return 0;
        }
        // 获取文件指针
        pos = getFilePointer();
        // 获取长度
        len = length();
        // 新的指针为原指针加上跳过的长度
        newpos = pos + n;
        // 当指针越界时，设置为长度
        if (newpos > len) {
            newpos = len;
        }
        // 设置文件偏移
        seek(newpos);
        // 返回实际移动的距离
        return (int) (newpos - pos);
    }
```

### write
```java
    // 写入字节数组
    public void write(byte b[]) throws IOException {
        // 直接调用native方法
        writeBytes(b, 0, b.length);
    }
```

### close
```java
    // 关闭
    public void close() throws IOException {
        // 进行同步操作，防止多个线程同时修改closed
        synchronized (closeLock) {
            if (closed) {
                return;
            }
            // volatile变量，保证可见行
            closed = true;
        }
        // 当channel不为null时，进行关闭
        if (channel != null) {
            channel.close();
        }
        // 关闭所有文件修饰符
        fd.closeAll(new Closeable() {
            public void close() throws IOException {
               close0();
           }
        });
    }
```
