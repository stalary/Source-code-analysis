<!-- GFM-TOC -->
* [常量](#常量)
* [put](#put)
<<<<<<< HEAD
* [get](#get)
=======
>>>>>>> 5235e778edcc1125bc55c512d95ea3bd1c0acdfb

### 常量
```java
// 数组的最大容量(少使用两次幂，前两位用于32位hash)
private static final int MAXIMUM_CAPACITY = 1 << 30;

// 默认初始化容量，必须是2的倍数，最大为MAXIMUM_CAPACITY
private static final int DEFAULT_CAPACITY = 16;

// 最大数组大小
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

// 表的默认并发级别，已经不使用，为了兼容以前的版本
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;

// 负载因子
private static final float LOAD_FACTOR = 0.75f;

// 链表转化为红黑树的阈值
static final int TREEIFY_THRESHOLD = 8;

// 红黑树转化为链表的阈值，扩容时才可能发生
static final int UNTREEIFY_THRESHOLD = 6;

// 进行树化的最小容量，防止在调整容量和形态是发生冲突
static final int MIN_TREEIFY_CAPACITY = 64;

// 作为下界避免遇到过多的内存争用
private static final int MIN_TRANSFER_STRIDE = 16;

// 用于sizeCtl产生标记的bit数量
private static int RESIZE_STAMP_BITS = 16;

// 可帮助调整的最大线程数
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

// sizeCtl移位大小标记
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

// 转移的hash
static final int MOVED     = -1; 

// 树根的hash
static final int TREEBIN   = -2; 

// ReservationNode的hash
static final int RESERVED  = -3; 

// 可用普通节点的hash
static final int HASH_BITS = 0x7fffffff; 

// 当前cpu可用的数量
static final int NCPU = Runtime.getRuntime().availableProcessors();
```

### put
```java
public V put(K key, V value) {
        // 直接调用putVal
        return putVal(key, value, false);
}
```

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        // 当key或者value为空时直接抛出空指针异常
        if (key == null || value == null) throw new NullPointerException();
        // 获取二次hash后的值
        int hash = spread(key.hashCode());
        // 操作次数
        int binCount = 0;
        // 死循环，直到插入成功
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            // 判断是否已经初始化，未初始化则进行初始化
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            // 获取该位置的节点，当为空时，没有发生碰撞，直接CAS进行存储，操作成功则退出死循环
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            // 为fh赋值，当检测到正在进行扩容，帮助扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                // 对Node(hash值相同的链表的头节点)上锁(1.7中锁segement)
                synchronized (f) {
                    // 双重检测
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            // 操作次数
                            binCount = 1;
                            // 死循环更新value，并增加操作数量
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                // 判断传入元素的hash和冲突节点的hash是否相同，key是否相同
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    // 将发生冲突的值赋值给oldVal
                                    oldVal = e.val;
                                    // putIfAbsent()方法中onlyIfAbsent为true
                                    if (!onlyIfAbsent)
                                        // 包含则赋值
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                // 查找下一个节点，并判断是否为空为，当为空时进行实例化
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        // 当为树型结构时
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            // 当向树型结构赋值成功时设置oldVal
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                // 当已经进行过节点赋值后，判断一下是否需要将链表转化为红黑树
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
}
```

### get

```java
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        // 获取二次hash后的
        int h = spread(key.hashCode());
        // 当所有节点不为空，并且能找到对应节点时进入操作否则直接返回null
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            // 获取头节点，判断hash值是否相等
            if ((eh = e.hash) == h) {
                // hash值相等时需要判断key是否相等(解决碰撞问题)
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            // 代表此时为树型结构，进行树的查找
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            // 遍历查找
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```




