* [介绍](#介绍)
* [常量](#常量)
* [put](#put)
* [get](#get)
* [resize](#resize)

### 介绍
HashMap是一个散列表，存储的内容是键值对(key-value)映射。它继承于AbstractMap，实现了Map、Cloneable、Serializable接口。HashMap 的实现不是同步的，这意味着它不是线程安全的。它的key、value都可以为null。此外，HashMap中的映射不是有序的。

### 常量
```java
// 默认初始容量必须是2的幂，这里是16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

// 最大容量（必须是2的幂且小于2的30次方，如果在构造函数中传入过大的容量参数将被这个值替换）
static final int MAXIMUM_CAPACITY = 1 << 30;

// 默认负载因子，啥叫负载因子呢......略
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 链表转化为红黑树的阈值
static final int TREEIFY_THRESHOLD = 8;

// 红黑树转化为链表的阈值，扩容时才可能发生
static final int UNTREEIFY_THRESHOLD = 6;

// 进行树化的最小容量，防止在调整容量和形态时发生冲突
static final int MIN_TREEIFY_CAPACITY = 64;
```

### put
```java
public V put(K key, V value) {
    // 求出key的hash值，并直接调用putVal
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 步骤一：如果table为空或length=0，则调用resize扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 步骤二：根据key的hash值得到插入的数组索引i，如果table[i]==null，直接新建节点并添加，转向步骤六
    // 如果table[i]不为空，转向步骤三
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 步骤三：判断table[i]的首个元素是否等于key，若相等直接覆盖value，若不相等转向步骤四
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 步骤四：判断table[i]是否为treeNode，即table[i]是否为红黑树，如果是红黑树直接在树中插入键值对，否则转向步骤五
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 步骤五：遍历table[i]
            for (int binCount = 0; ; ++binCount) {
                // 若链表长度小于8，执行链表的插入操作
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 若链表长度大于等于8，将链表转换为红黑树并执行插入操作
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 在遍历过程中，若发现key已经存在则直接覆盖value
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 若key已经存在，用新的value替换原先的value，并将原先的value返回
        if (e != null) { 
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    // modCount+1，modCount用于实现fail-fast机制，啥叫fail-fast呢......略
    ++modCount;
    // 步骤六：插入成功后，判断实际存在的键值对数量size是否超过最大容量threshold，如果超过则调用resize扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

### get
```java
public V get(Object key) {
    Node<K,V> e;
    // 求出key的hash值，并直接调用getNode
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 当table不为空，并且经过计算得到的插入位置table[i]也不为空时继续操作，否则返回null
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 判断table[i]的首个元素是否等于key，若相等将其返回
        if (first.hash == hash &&
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // 若存储结构为红黑树，则执行红黑树中的查找操作
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                // 若存储结构仍为链表，则遍历链表，找到key所在的位置
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### resize
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 如果扩容前的数组大小超过最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 修改resize阈值为int的最大值(2^31-1)，这样以后就不会扩容了
            threshold = Integer.MAX_VALUE;
            // 直接将原数组返回
            return oldTab;
        }
        // 没超过最大容量，就将容量扩充为原来的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 将新的resize阈值也扩充为原来的两倍
            newThr = oldThr << 1;
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {
                         // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize阈值
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        // 使用resize后的容量新建一个空的table
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 
    if (oldTab != null) {
        // 遍历旧的table
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                // 释放旧table的对象引用（for循环后，旧的table不再引用任何对象）
                oldTab[j] = null;
                // 若oldTab[j]只包含一个元素
                if (e.next == null)
                    // 直接将这一个元素放到newTab合适的位置
                    newTab[e.hash & (newCap - 1)] = e;
                // 若oldTab[j]存储结构为红黑树，执行红黑树中的调整操作
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    // 若oldTab[j]存储结构为链表
                    // 这一波操作比较巧妙，与JDK 1.7相比，既不需要重新计算hash，也避免了链表元素倒置的情况
                    // 由于比较巧妙，还在研究当中......研究透了再补充
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}