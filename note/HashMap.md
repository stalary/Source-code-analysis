* [介绍](#介绍)
* [常量](#常量)
* [put](#put)

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
    // modCount+1，modCount用于实现fail-fast机制，啥叫fail-fast......略
    ++modCount;
    // 步骤六：插入成功后，判断实际存在的键值对数量size是否超过最大容量threshold，如果超过则调用resize扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```