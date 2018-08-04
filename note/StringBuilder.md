- [介绍](#%E4%BB%8B%E7%BB%8D)
- [常量](#%E5%B8%B8%E9%87%8F)
- [append](#append)
- [delete](#%08delete)
- [insert](#insert)
- [reverse](#reverse)
- [toString](#tostring)
### 介绍
- 一个可变的字符序列
- 线程不安全
- StringBuffer就是把StringBuilder的方法都加上了synchronized，不再赘述

### 常量
```java
// 继承自AbstractStringBuilder，用于存储字符值
char[] value;

// 统计所使用的字符数量
int count;
```

### append
```java
    // 添加一个字符串
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
        // 求出传入字符串的长度
        int len = str.length();
        // 进行扩容
        ensureCapacityInternal(count + len);
        // 写入字符串
        str.getChars(0, len, value, count);
        // 长度增加
        count += len;
        return this;
    }
```

```java
    private void ensureCapacityInternal(int minimumCapacity) {
        // overflow-conscious code
        // 当容量扩大时，进行扩容复制
        if (minimumCapacity - value.length > 0) {
            value = Arrays.copyOf(value,
                    newCapacity(minimumCapacity));
        }
    }
```

```java
    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        // 扩容为原来的两倍+2
        int newCapacity = (value.length << 1) + 2;
        // 当新容量比传入容量小时，新容量赋值为传入容量
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        // 当新容量的值小于等于0或者大于最大数组容量时进入hugeCapacity
        return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
            ? hugeCapacity(minCapacity)
            : newCapacity;
    }
```

```java
    private int hugeCapacity(int minCapacity) {
        // 当容量已经超过最大值时，直接抛出
        if (Integer.MAX_VALUE - minCapacity < 0) { // overflow
            throw new OutOfMemoryError();
        }
        // 否则返回传入值和最大数组大小的最大值
        return (minCapacity > MAX_ARRAY_SIZE)
            ? minCapacity : MAX_ARRAY_SIZE;
    }
```

### delete
```java
    // 删除某一范围内的字符
    public StringBuilder delete(int start, int end) {
        // 直接调用父类的delete
        super.delete(start, end);
        return this;
    }
```

```java
    public AbstractStringBuilder delete(int start, int end) {
        // 边界检测
        if (start < 0)
            throw new StringIndexOutOfBoundsException(start);
        if (end > count)
            end = count;
        if (start > end)
            throw new StringIndexOutOfBoundsException();
        // 求出删除的长度
        int len = end - start;
        if (len > 0) {
            // 复制数组，减去删除的字符
            System.arraycopy(value, start+len, value, start, count-end);
            // 修改长度
            count -= len;
        }
        return this;
    }
```

```java
    // 删除指定下标的元素
    public StringBuilder deleteCharAt(int index) {
        // 直接调用父类的方法
        super.deleteCharAt(index);
        return this;
    }
```

```java
    // 与delete原理相同
    public AbstractStringBuilder deleteCharAt(int index) {
        // 边界检测
        if ((index < 0) || (index >= count))
            throw new StringIndexOutOfBoundsException(index);
        // 复制元素
        System.arraycopy(value, index+1, value, index, count-index-1);
        // 修改长度
        count--;
        return this;
    }
```

### insert
```java
    // 向某一位置插入字符串
    public StringBuilder insert(int offset, String str) {
        // 直接调用父类方法
        super.insert(offset, str);
        return this;
    }
```

```java
    public AbstractStringBuilder insert(int offset, String str) {
        // 边界检测
        if ((offset < 0) || (offset > length()))
            throw new StringIndexOutOfBoundsException(offset);
        // 传入字符串为null，转化为字符串的"null"
        if (str == null)
            str = "null";
        // 计算传入字符串的长度
        int len = str.length();
        // 扩容
        ensureCapacityInternal(count + len);
        // 将元素后移
        System.arraycopy(value, offset, value, offset + len, count - offset);
        // 插入元素
        str.getChars(value, offset);
        // 修改元素数量
        count += len;
        return this;
    }
```

### reverse
```java
    // 反转字符串
    public StringBuilder reverse() {
        // 直接调用父类
        super.reverse();
        return this;
    }
```

```java
    public AbstractStringBuilder reverse() {
        boolean hasSurrogates = false;
        int n = count - 1;
        // 从中间位置开始交换
        for (int j = (n-1) >> 1; j >= 0; j--) {
            // 后半部分元素
            int k = n - j;
            // 交换
            char cj = value[j];
            char ck = value[k];
            value[j] = ck;
            value[k] = cj;
            // 判断是否是两单元存储的unicode
            if (Character.isSurrogate(cj) ||
                Character.isSurrogate(ck)) {
                hasSurrogates = true;
            }
        }
        if (hasSurrogates) {
            reverseAllValidSurrogatePairs();
        }
        return this;
    }
```

### toString
```java
    // 转化为String字符串
    public String toString() {
        // Create a copy, don't share the array
        // 直接生成一个新的副本,这里与StringBuffer有不同，StringBuffer会首先判断是否有缓存，没有缓存就复制到缓存，最后将缓存进行共享
        /**
        if (toStringCache == null) {
            toStringCache = Arrays.copyOfRange(value, 0, count);
        }
        return new String(toStringCache, true);
        **/
        return new String(value, 0, count);
    }
```




