<!-- GFM-TOC -->
* [介绍](#介绍)
* [常量](#常量)
* [put](#put)
* [get](#get)
* [transfer](#transfer)
* [tryPresize](#tryPresize)

### 介绍
一个支持并发查找和并发修改的hash表，方法与Hashtable一致，但是没有锁定整个hash表

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

### transfer

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        // 判断当前可用线程是否大于1，大于1时则进行并行操作
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        if (nextTab == null) {            // initiating
            try {
                // 构造一个原来容量两倍的对象
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        int nextn = nextTab.length;
        // 正在迁移的Node，该节点hash为MOVED，作为标志使用
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        // 表示是否已经完成迁移
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
        // i位置索引，bound边界
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            while (advance) {
                int nextIndex, nextBound;
                // 判断是否已经处理过
                if (--i >= bound || finishing)
                    advance = false;
                // 原数组的所有位置都有相应的线程去处理
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    // 赋值迁移边界                   
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                // 迁移工作完成
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                // 使用cas来修改数量，代表完成当前任务
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    // 所有任务都完成
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            // 如果位置i是空的，没有任何节点，放入刚刚实例化的 ForwardingNode
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            // 该位置是ForwardingNode，代表已经完成过迁移
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                // 对当前位置节点加锁
                synchronized (f) {
                    // 获取头节点
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        // 头节点hash>=0代表为链表
                        if (fh >= 0) {
                            // 将链表进行划分，分成两部分进行迁移
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            // 分别存入两个链表中
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            // 其中的一个链表放在新数组的i位置
                            setTabAt(nextTab, i, ln);
                            // 另一个链表放在新数组的i+n位置(n为原长度)
                            setTabAt(nextTab, i + n, hn);
                            // 将原数组该位置处设置为 fwd，代表该位置已经处理完毕
                            setTabAt(tab, i, fwd);
                            // 迁移完成
                            advance = true;
                        }
                        // 当为红黑树时，开始树型迁移
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            // 进行划分，分为两部分迁移
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            // 如果一分为二后，节点数少于8，那么将红黑树转换回链表
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            // 其中的一部分放在新数组的i位置
                            setTabAt(nextTab, i, ln);
                            // 另一部分放在新数组的i+n位置
                            setTabAt(nextTab, i + n, hn);
                            // 将原数组该位置处设置为 fwd，代表该位置已经处理完毕
                            setTabAt(tab, i, fwd);
                            // 迁移完毕
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```

### tryPresize

```java
private final void tryPresize(int size) {
        // 如果大小已经大于等于最大容量的一半，直接扩容到最大容量，否则*1.5倍+1并且向上取到二次幂
        int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
            tableSizeFor(size + (size >>> 1) + 1);
        int sc;
        while ((sc = sizeCtl) >= 0) {
            Node<K,V>[] tab = table; int n;
            // 当节点还未初始化时
            if (tab == null || (n = tab.length) == 0) {
                // 取sc和c的较大值
                n = (sc > c) ? sc : c;
                // 通过cas修改SIZECTL为-1，表示正在初始化
                if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                    try {
                        // 判断是否没有被其他线程修改
                        if (table == tab) {
                            @SuppressWarnings("unchecked")
                            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                            table = nt;
                            // 0.75*n
                            sc = n - (n >>> 2);
                        }
                    } finally {
                        // 最后将sizeCtl设置为sc
                        sizeCtl = sc;
                    }
                }
            }
            // 如果扩容大小没有达到阈值，或者超过最大容量时退出
            else if (c <= sc || n >= MAXIMUM_CAPACITY)
                break;
            else if (tab == table) {
                // 重新生成戳
                int rs = resizeStamp(n);
                // 当线程在进行扩容时
                if (sc < 0) {
                    Node<K,V>[] nt;
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                // 未进行扩容时，cas修改sizeCtl值
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
            }
        }
    }
```



