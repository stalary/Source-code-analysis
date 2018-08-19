- [介绍](#介绍)
- [field](#field)
- [constructor](#constructor)
- [getEncode](#getencode)
- [read](#read)
- [ready](#ready)
- [close](#close)


### 介绍
InputStreamReader继承了Reader。他是字节流到字符流的桥梁，
读取字节并用特定的字符集解码为字符，字符集可以显式指定否则就使用平台默认的。要保证效率，就用BufferedReader。

> BufferedReader in = new BufferedReader(new InputStreamReader(System.in));

### field

```java

//用于decode，大部分操作都是靠他实现的，装饰器模式
private final StreamDecoder sd;

```

### constructor

```java

//默认charset
public InputStreamReader(InputStream in);

//指定charset名
public InputStreamReader(InputStream in, String charsetName)
    throws UnsupportedEncodingException;
    
//指定Charset
public InputStreamReader(InputStream in, Charset cs);

//指定CharsetDecoder
public InputStreamReader(InputStream in, CharsetDecoder dec);


```

### getEncode

```java

//获取编码方式
public String getEncoding() {
    return sd.getEncoding();
}

```

### read

```java

//读一个字符
public int read() throws IOException {
    return sd.read();
}

//读到cbuf中，offset为开始存储的位置，length为最大读取长度
public int read(char cbuf[], int offset, int length) throws IOException {
    return sd.read(cbuf, offset, length);
}

```

### ready

```java

//是否可读
public boolean ready() throws IOException {
    return sd.ready();
}


```

### close

```java
//关闭流
public void close() throws IOException {
    sd.close();
}

```


