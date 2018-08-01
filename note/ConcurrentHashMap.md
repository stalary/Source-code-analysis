<!-- GFM-TOC -->
* [常量](#常量)
* [put](#put)

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
        // 进行二次hash，减少hash冲突
        int hash = spread(key.hashCode());
        // 操作次数
        int binCount = 0;
        // 死循环，直到插入成功
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            // 判断是否已经初始化，未初始化则进行初始化
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            // 该位置节点为空时，没有发生碰撞，直接CAS进行存储，操作成功则退出死循环
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            // 检测到正在进行扩容，帮助扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                // 对Node上锁(1.7中锁segement)
                synchronized (f) {
                    // 
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
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





