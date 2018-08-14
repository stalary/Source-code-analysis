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
        // 只读方式打开
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
        // 安全管理器，检测操作是否合法
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
        // 文件非法，抛出非法路径异常
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
