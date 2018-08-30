- [介绍](#介绍)
- [常量&变量](#常量变量)
- [load](#load)
- [store](#store)
- [getProperty](#getproperty)
- [setProperty](#setproperty)
- [list](#list)


### 介绍

Properties类抽象了一个持久的属性集，可以被存储到一个流中或从一个流中加载。Properties里面的key和value都是String类型。Properites继承自HashTable，但是要避免使用put和putAll方法，防止放入非String类型的数据，应该使用setProperty方法

一个属性集可以包含另一个属性集作为他的默认属性，当一个属性键在原始的Properties中找不着时，才会搜索这个默认属性集。

Properties还支持对xml的读取和存储。

### 常量&变量

```java

// 默认属性集
protected Properties defaults;

//十六进制数常量
private static final char[] hexDigit = {
    '0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'
};

```

### load

```java


//将流封装成Properties.LineReader，然后用load0读取properties，LineReader是面向行读取的Reder
public synchronized void load(Reader reader) throws IOException {
    load0(new LineReader(reader));
}

public synchronized void load(InputStream inStream) throws IOException {
    load0(new LineReader(inStream));
}


/*
行分为自然行和逻辑行。自然行被定义为以("\n","\r","\r\n"，eof)任一结尾的字符行。
自然行也可以是空行，注释行(#,!作为第一个非空白字符的行)，或者保存了全部或部分键-元素对的字符行。
逻辑行保存了所有键-元素对的数据，可能分散在多个相邻的自然行中，用反斜杠字符 \ 转义行结束符序列。

key和value中间的分隔符可以是":"、"="，" "，分隔符左右两侧的空格会被删掉，key前的空格也会被去掉
*/

 private void load0(LineReader lr) throws IOException {
    char[] convtBuf = new char[1024];
    int limit;  // 字符总数
    int keyLen; // key的长度
    int valueStart; // value的起始位置
    char c;
    boolean hasSep;
    boolean precedingBackslash; // 是否是转义字符

    while ((limit = lr.readLine()) >= 0) {
        c = 0;
        keyLen = 0;
        // value的起始位置默认为limit
        valueStart = limit;
        hasSep = false;
        precedingBackslash = false;

        // keyLen < limit
        while (keyLen < limit) {
            c = lr.lineBuf[keyLen];
            // 检测到非空格分隔符且前面的字符没有转义
            if ((c == '=' || c == ':') && !precedingBackslash) {
                //下一个就是value开始的位置
                valueStart = keyLen + 1;
                // 并且指定，去除空格
                hasSep = true;
                break;
            } else if ((c == ' ' || c == '\t' || c == '\f') && !precedingBackslash) {
                // 检测到空格分隔符
                valueStart = keyLen + 1;
                break;
            }
            // 检测到'\'，记录
            if (c == '\\') {
                precedingBackslash = !precedingBackslash;
            } else {
                precedingBackslash = false;
            }
            //前进一位
            keyLen++;
        }

        // valueStart < limit
        while (valueStart < limit) 
            c = lr.lineBuf[valueStart];
            // 判断是否是空格类字符
            if (c != ' ' && c != '\t' && c != '\f') {
                // 不是空格类字符，并且第一次出现非空格分隔符
                if (!hasSep && (c == '=' || c == ':')) {
                    hasSep = true;
                } else {
                    break;
                }
            }
            valueStart++;
        }
        // 读取key和value
        String key = loadConvert(lr.lineBuf, 0, keyLen, convtBuf);
        String value = loadConvert(lr.lineBuf, valueStart, limit - valueStart, convtBuf);
        // 放入
        put(key, value);
    }
}


```

### store

```java
// 存储，给定Writer或OutStreamWriter和comments
public void store(Writer writer, String comments)
    throws IOException
{
    store0((writer instanceof BufferedWriter)?(BufferedWriter)writer
     : new BufferedWriter(writer),comments,false);
}



private void store0(BufferedWriter bw, String comments, boolean escUnicode) throws IOException {
        if (comments != null) {
            // 写入注释,用 8859-1存储中文
            writeComments(bw, comments);
        }
        // 写入时间
        bw.write("#" + new Date().toString());
        //另起一行
        bw.newLine();
        // 同步
        synchronized (this) {
            for (Enumeration e = keys(); e.hasMoreElements();) {
                String key = (String) e.nextElement();
                String val = (String) get(key);
                // 对key中的空格转义
                key = saveConvert(key, true, escUnicode);
                // 不转义value的的空格
                val = saveConvert(val, false, escUnicode);
                // 写入按照key=value
                bw.write(key + "=" + val);
                bw.newLine();
            }
        }
        bw.flush();
    }

```


### getProperty

```java

//获取属性
public String getProperty(String key) {
    Object oval = super.get(key
    //存储的不是字符串就返回null
    String sval = (oval instanceof String) ? (String)oval : null;
    //如果默认属性中存在就返回
    return ((sval == null) && (defaults != null)) ? defaults.getProperty(key) : sval;
}
//给定默认值
public String getProperty(String key, String defaultValue) {
    String val = getProperty(key);
    //默认属性集的优先级要高于给定的默认值
    return (val == null) ? defaultValue : val;
}

```

### setProperty

```java

// 同步方法
public synchronized Object setProperty(String key, String value) {
    return put(key, value);
}

```


### list

```java

//打印属性集
public void list(PrintWriter out);
public void list(PrintStream out);

//exm：直接打印到标准输出流
properties.list(System.out);

```

