- [介绍](#%E4%BB%8B%E7%BB%8D)
- [方法](#%E6%96%B9%E6%B3%95)
### 介绍
- 用于获取资源
- 继承了InputStreamSource
  
### 方法
```java
// 判断资源是否存在
boolean exists();

// 判断资源是否为可读的
default boolean isReadable() {
	return true;
}

// 判断资源是否已经被打开
default boolean isOpen() {
	return false;
}

// 判断资源是否为文件
default boolean isFile() {
	return false;
}

// 获取资源对应的url
URL getURL() throws IOException;

// 获取资源对应的uri
URI getURI() throws IOException;

// 获取资源对应的文件
File getFile() throws IOException;

// 返回InputStream对应的Channel
default ReadableByteChannel readableChannel() throws IOException {
	return Channels.newChannel(getInputStream());
}

// 返回内容的长度
long contentLength() throws IOException;

// 返回上次修改的时间戳
long lastModified() throws IOException;

// 按照相对路径创建一个资源
Resource createRelative(String relativePath) throws IOException;

// 获取资源所对应的文件名称
String getFilename();

// 获取资源对应的描述
String getDescription();
```