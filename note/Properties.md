### 介绍

Properties类抽象了一个持久的属性集，可以被存储到一个流中或从一个流中加载。Properties里面的key和value都是String类型。Properites继承自HashTable，但是要避免使用put和putAll方法，防止放入非String类型的数据，应该使用setProperty方法

一个属性集可以包含另一个属性集作为他的默认，当一个属性键在原始的Properties中找不着时，才会搜索这个默认属性集。

### const&field

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


//从Properties.LineReader中读取properties，LineReader是面向行读取的Reder
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
*/

private void load0 (LineReader lr) throws IOException {
    char[] convtBuf = new char[1024];
    int limit;
    int keyLen;
    int valueStart;
    char c;
    boolean hasSep;
    boolean precedingBackslash;

    while ((limit = lr.readLine()) >= 0) {
        c = 0;
        keyLen = 0;
        valueStart = limit;
        hasSep = false;

        precedingBackslash = false;
        while (keyLen < limit) {
            c = lr.lineBuf[keyLen];
            //需要检查是否转义
            if ((c == '=' ||  c == ':') && !precedingBackslash) {
                valueStart = keyLen + 1;
                hasSep = true;
                break;
            } else if ((c == ' ' || c == '\t' ||  c == '\f') && !precedingBackslash) {
                valueStart = keyLen + 1;
                break;
            }
            if (c == '\\') {
                precedingBackslash = !precedingBackslash;
            } else {
                precedingBackslash = false;
            }
            keyLen++;
        }
        while (valueStart < limit) {
            c = lr.lineBuf[valueStart];
            if (c != ' ' && c != '\t' &&  c != '\f') {
                if (!hasSep && (c == '=' ||  c == ':')) {
                    hasSep = true;
                } else {
                    break;
                }
            }
            valueStart++;
        }
        String key = loadConvert(lr.lineBuf, 0, keyLen, convtBuf);
        String value = loadConvert(lr.lineBuf, valueStart, limit - valueStart, convtBuf);
        put(key, value);
    }
}


```