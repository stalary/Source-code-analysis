- [介绍](#介绍)
- [常量](#常量)
- [构造](#构造)
- [is](#is)
- [get](#get)
- [list](#list)
- [toString](#tostring)
- [其他](#其他)


### 介绍
- File类

File类抽象了文件和目录，其封装的并不是一个真正存在的文件，实现了Serializable, Comparable<File>接口，主要提供文件和目录的创建、文件查找和删除、获取文件信息、文件权限相关等操作。File对象代表磁盘中实际存在的文件和目录。

- 关于抽象路径

绝对路径和相对路径都是平台相关的，构造函数会将相对和绝对路径转换为平台无关的路径，即抽象路径。

抽象路径名转换成路径名时候，相邻的两个名字中间加入一个系统默认的分隔符。抽象路径名转换成路径名时候，路径名中的名字会被默认的分隔符，或者是底层系统支持的任意的分隔符分割。抽象路径中的分隔符没有意义，转为路径名会根据对应平台转换的，以实现平台无关。

在C:\Idea\src\main中Idea、src、main都是指“名字”，而C:\则是前缀

抽象路径名所代表的文件或目录既可以存在也可以不存在。比如如果我们new了一个不存在的目录，并不会去创建他，除非调用mkdir()



### 常量

```

/*
代表平台的本地文件系统的FileSystem对象。
FileSystem只是抽象类，真正操作的是他的实现类，
在win下，是WinNTFileSystem
在Linux下是UnixFileSystem
*/
private static final FileSystem fs = DefaultFileSystem.getFileSystem();

//抽象路径名标准化了的路径名，使用默认的分隔符，不含多余的分隔符
private final String path;

//前缀长度
private final transient int prefixLength;

//从系统配置中获取的平台文件分隔符
public static final char separatorChar = fs.getSeparator();

```

### 构造

```

//给定路径名，转换为抽象路径名
public File(String pathname) {
    if (pathname == null) {
        throw new NullPointerException();
    }
    this.path = fs.normalize(pathname);
    this.prefixLength = fs.prefixLength(this.path);
}

//给出父抽象路径名和子路径字符串，进行构造
public File(File parent, String child) {
    if (child == null) {
        throw new NullPointerException();
    }
    
    if (parent != null) {
        //父路径为空串
        if (parent.path.equals("")) {
            //根据默认的父路径解析
            this.path = fs.resolve(fs.getDefaultParent(),
                                   fs.normalize(child));
        } else {
            //加入父路径解析
            this.path = fs.resolve(parent.path,
                                   fs.normalize(child));
        }
    } else {
        //父路径不存在，相当于File(String)
        this.path = fs.normalize(child);
    }
    this.prefixLength = fs.prefixLength(this.path);
}

//给定父路径字符串和子路径字符串，类似上面
public File(String parent, String child) {
    if (child == null) {
        throw new NullPointerException();
    }
    if (parent != null) {
        if (parent.equals("")) {
            this.path = fs.resolve(fs.getDefaultParent(),
                                   fs.normalize(child));
        } else {
            this.path = fs.resolve(fs.normalize(parent),
                                   fs.normalize(child));
        }
    } else {
        this.path = fs.normalize(child);
    }
    this.prefixLength = fs.prefixLength(this.path);
}

//用于file:这种
public File(URI uri);


```

### is

```

//检查抽象路径是不是个绝对路径
public boolean isAbsolute() {
    return fs.isAbsolute(this);
}

//***主要根据FileSystem里定义的常量判断***

//是否存在该目录/文件
public boolean exists() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkRead(path);
    }
    if (isInvalid()) {
        return false;
    }
    return ((fs.getBooleanAttributes(this) & FileSystem.BA_EXISTS) != 0);
}

//是否目录
public boolean isDirectory();

//是否文件
public boolean isFile();

//是否隐藏
public boolean isHidden();

```


### get

```
//获取绝对路径，解析由FileSystem的具体实现类实现
public String getAbsolutePath() {
    return fs.resolve(this);
}

//返回由此抽象路径名表示的文件或目录的名称，仅仅是目录或文件名
public String getName() {
    //查找最后一个分隔符
    int index = path.lastIndexOf(separatorChar);
    if (index < prefixLength) return path.substring(prefixLength);
    return path.substring(index + 1);
}

// 返回此抽象路径名的父路径名的路径名字符串，
// 如果此路径名没有指定父目录，则返回 null。
public String getParent() {
    int index = path.lastIndexOf(separatorChar);
    if (index < prefixLength) {
        if ((prefixLength > 0) && (path.length() > prefixLength))
            return path.substring(0, prefixLength);
        return null;
    }
    return path.substring(0, index);
}

//获取绝对路径
public String getAbsolutePath() {
    return fs.resolve(this);
}

//相当于new File(this.getAbsolutePath())
public File getAbsoluteFile() {
    String absPath = getAbsolutePath();
    return new File(absPath, fs.prefixLength(absPath));
}




```

### list 

```

//返回目录下面的path
public String[] list() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkRead(path);
    }
    if (isInvalid()) {
        return null;
    }
    return fs.list(this);
}

//根据Filter进行过滤，需要自己实现FilenameFilter接口
public String[] list(FilenameFilter filter) {
    String names[] = list();
    if ((names == null) || (filter == null)) {
        return names;
    }
    List<String> v = new ArrayList<>();
    for (int i = 0 ; i < names.length ; i++) {
        if (filter.accept(this, names[i])) {
            v.add(names[i]);
        }
    }
    return v.toArray(new String[v.size()]);
}
```


### toString

```
//可以看到，直接返回了path
public String toString() {
    return getPath();
}

```

### 其他

```
//创建一个目录
public boolean mkdir() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkWrite(path);
    }
    if (isInvalid()) {
        return false;
    }
    return fs.createDirectory(this);
}

//该路径名所抽象的File不存在的话，创建一个空文件
public boolean createNewFile() throws IOException {
    SecurityManager security = System.getSecurityManager();
    if (security != null) security.checkWrite(path);
    if (isInvalid()) {
        throw new IOException("Invalid file path");
    }
    return fs.createFileExclusively(path);
}

//删除目录或文件，如果是目录，必须保证为空
public boolean delete() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkDelete(path);
    }
    if (isInvalid()) {
        return false;
    }
    return fs.delete(this);
}


```