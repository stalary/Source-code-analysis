* [介绍](#介绍)
* [常量](#常量)

### 介绍
- 一个可变的字符序列
- 线程不安全

### 常量
```java
// 继承自AbstractStringBuilder，用于存储字符值
char[] value;

// 统计所使用的字符数量
int count;
```

### append
```java
    public StringBuilder append(String str) {
        // 直接调用父类的append
        super.append(str);
        return this;
    }
```

```java
    public AbstractStringBuilder append(String str) {
        // 当传入的字符串为null时，直接转化为null字符串存入
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
```




