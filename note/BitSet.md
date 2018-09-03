- [介绍](#介绍)
- [const&field](constfield)
- [构造](#构造)
- [valueOf](valueof)
- [get](#get)
- [set](#set)
- [expandTo&ensureCapacity](#expandtoensurecapacity)
- [flip](#flip)
- [nextSetBit&nextClearBit](#nextSetBitnextClearBit)
- [and&or&xor](#andorxor)
- [长度和容量](#长度和容量)

### 介绍

- 可以看做操作位的Vector，容量可以动态变化。
- 实现了Cloneable和Serializable接口。
- 里面的位默认都是false。
- 一个BitSet可以通过逻辑AND、OR、XOR修改其他BitSet的内容
- java.util下的，实际上不属于集合框架，但是和List比较像
- 线程不安全


### const&field

```java

private final static int ADDRESS_BITS_PER_WORD = 6;

// 每个word的bit数 = 2^6 = 64个位，对应8字节的long类型
private final static int BITS_PER_WORD = 1 <<
ADDRESS_BITS_PER_WORD;

// 掩码，BIT_PER_WORD - 1 每一个有效位都是1
private final static int BIT_INDEX_MASK = BITS_PER_WORD - 1;

// long类型的掩码 16^16
private static final long WORD_MASK = 0xffffffffffffffffL;

// 内部存储"bit"的空间
private long[] words;

// word数
private transient int wordsInUse = 0;

// words的size是否由用户指定
private transient boolean sizeIsSticky = false;


```

### 构造

```java

// 默认
public BitSet() {
    initWords(BITS_PER_WORD);
    sizeIsSticky = false;
}

 // nbits为至少要存的位的个数
public BitSet(int nbits) {
    if (nbits < 0)
        throw new NegativeArraySizeException("nbits < 0: " + nbits);

    initWords(nbits);
    sizeIsSticky = true;
}

// 给定内部的words数组，最后一个word不能是0
private BitSet(long[] words) {
    this.words = words;
    this.wordsInUse = words.length;
    checkInvariants();
}


```

举例：
- 1个long有 1 << 6 个位
- 给定 nbits = 127，(127 - 1) >> 6 = 1，那么要填满1个long，2个long肯定没问题
- 给定 nbits = 129，(129 - 1) >> 6 = 2，至少要填满2个long，3个long肯定没问题 

```java

// 开辟一块内存至少要存下nbits个位
private void initWords(int nbits) {
    words = new long[wordIndex(nbits-1) + 1];
}

// 计算需要填满几个long
private static int wordIndex(int bitIndex) {  
    return bitIndex >> ADDRESS_BITS_PER_WORD;  
}  


```


### valueOf

```java

// 给定数据，构造返回一个BitSet
public static BitSet valueOf(long[] longs);

public static BitSet valueOf(LongBuffer lb);

public static BitSet valueOf(byte[] bytes);

public static BitSet valueOf(ByteBuffer bb);

```

### get

```java

public boolean get(int bitIndex) {
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);
    checkInvariants();
    // 定位
    int wordIndex = wordIndex(bitIndex);
    // 使用与操作判断
    return (wordIndex < wordsInUse)
        && ((words[wordIndex] & (1L << bitIndex)) != 0);
}


```

### set

set用于设置给定位置的位，默认设置为true，也可以指定。
支持范围fromIndex到toIndex。

```java


public void set(int bitIndex) {
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);
    // 定位到long[]中的位置
    int wordIndex = wordIndex(bitIndex);
    // 扩容操作
    expandTo(wordIndex);
    // 不管是不是1，结果都是1
    words[wordIndex] |= (1L << bitIndex); 
    checkInvariants();
}

public void set(int bitIndex, boolean value) {
    if (value)
        set(bitIndex);
    else
        // 设为0
        clear(bitIndex);
}

// 范围操作
public void set(int fromIndex, int toIndex) {
    checkRange(fromIndex, toIndex);
    // 直接退出
    if (fromIndex == toIndex)
        return;
        
    // 定位
    int startWordIndex = wordIndex(fromIndex);
    int endWordIndex   = wordIndex(toIndex - 1);
    // 保证容量
    expandTo(endWordIndex);
    // 计算掩码
    long firstWordMask = WORD_MASK << fromIndex;
    long lastWordMask  = WORD_MASK >>> -toIndex;
    // 都在同一个word范围内
    if (startWordIndex == endWordIndex) {
        // 两个掩码与运算，然后或运算
        words[startWordIndex] |= (firstWordMask & lastWordMask);
    } else {
        // 否则 两端对掩码进行或操作，中间置为1
        
        // 开始
        words[startWordIndex] |= firstWordMask;

        // 中间全设为1
        for (int i = startWordIndex+1; i < endWordIndex; i++)
            words[i] = WORD_MASK;
        
        // 结束
        words[endWordIndex] |= lastWordMask;
    }

    checkInvariants();
}

```

### expandTo&ensureCapacity

```java

// 扩容
private void expandTo(int wordIndex) {
    int wordsRequired = wordIndex+1;
    if (wordsInUse < wordsRequired) {
        ensureCapacity(wordsRequired);
        wordsInUse = wordsRequired;
    }
}

// 保证空间够用
private void ensureCapacity(int wordsRequired) {
    // 小于实际所需
    if (words.length < wordsRequired) {
        // 2倍扩容或扩容到实际长度，两者取其大
        int request = Math.max(2 * words.length, wordsRequired);
        words = Arrays.copyOf(words, request);
        sizeIsSticky = false;
    }
}


```

### flip

反转该位，1设为0，0设为1，借用xor运算

```java

public void flip(int bitIndex) {
    if (bitIndex < 0)
        throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);

    int wordIndex = wordIndex(bitIndex);
    expandTo(wordIndex);
    
    words[wordIndex] ^= (1L << bitIndex);

    recalculateWordsInUse();
    checkInvariants();
}

```
### nextSetBit&nextClearBit
这两个方法分别用于找给定索引位置之后的下一个1(Set)或者0(Clear), 这里只贴一个nextSetBit

```java
public int nextSetBit(int fromIndex) {
    if (fromIndex < 0)
        throw new IndexOutOfBoundsException("fromIndex < 0: " + fromIndex);

    checkInvariants();

    int u = wordIndex(fromIndex);
    // 超出范围肯定没有了
    if (u >= wordsInUse)
        return -1;

    long word = words[u] & (WORD_MASK << fromIndex);
    // 依次寻找
    while (true) {
        if (word != 0)
            //带上前面的
            return (u * BITS_PER_WORD) + Long.numberOfTrailingZeros(word);
        if (++u == wordsInUse)
            return -1;
        word = words[u];
    }
}



```

### and&or&xor

and、or、xor的实现类似，只贴一个and。操作的结果放在本对象里。

```java

public void and(BitSet set) {
    if (this == set)
        return;
    
    // 多出来的都设成0
    while (wordsInUse > set.wordsInUse)
        words[--wordsInUse] = 0;

    // 依次and操作
    for (int i = 0; i < wordsInUse; i++)
        words[i] &= set.words[i];
    
    // 重新计算wordsInUse
    recalculateWordsInUse();
    checkInvariants();
}


```


### 长度和容量

```java

public int size() {
    // 所以说这里返回的所占的空间能存储的bit位数，BITS_PER_WORD的倍数
    return words.length * BITS_PER_WORD;
}

public int length() {
    if (wordsInUse == 0)
        return 0;
    // 返回实际有效的，减去后面的0位
    return BITS_PER_WORD * (wordsInUse - 1) +
        (BITS_PER_WORD - Long.numberOfLeadingZeros(words[wordsInUse - 1]));
}

// 返回1的个数
public int cardinality() {
    int sum = 0;
    for (int i = 0; i < wordsInUse; i++)
        sum += Long.bitCount(words[i]);
    return sum;
}

```