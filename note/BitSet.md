### 介绍

- 可以看做操作位的Vector，容量可以动态变化。
- 实现了Cloneable和Serializable接口。
- 里面的位默认都是false。
- 一个BitSet可以通过逻辑AND、OR、XOR修改其他BitSet的内容
- 线程不安全
- java.util下的，实际上不属于集合框架，但是和List比较像

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

//默认
public BitSet() {
    initWords(BITS_PER_WORD);
    sizeIsSticky = false;
}

 //nbits为至少要存的位的个数
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

//计算需要填满几个long
private static int wordIndex(int bitIndex) {  
    return bitIndex >> ADDRESS_BITS_PER_WORD;  
}  

```