- [介绍](#%E4%BB%8B%E7%BB%8D)
- [常量&变量](#%E5%B8%B8%E9%87%8F%E5%8F%98%E9%87%8F)
- [添加](#%E6%B7%BB%E5%8A%A0)
- [查找](#%E6%9F%A5%E6%89%BE)
- [success后继](#success%E5%90%8E%E7%BB%A7)

### 介绍

- 如果我们希望Map可以保持key的大小顺序时，就需要利用TreeMap。

- 底层使用了红黑树，左子树总小于root，右子树总大于root，具有很好的平衡性,操作速度达到log(n)。

### 常量&变量
```java
    
    //定义key的排序规则
    private final Comparator<? super K> comparator;

	//根节点
    private transient Entry<K,V> root;

    //元素个数
    private transient int size = 0;

    //一次操作所修改的元素个数
    private transient int modCount = 0;
       
```
### 添加

```java
//如果key存在的话，old value被替换，否则新建一个节点，然后做红黑树的平衡操作
public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {
            compare(key, key); // type (and possibly null) check

            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        //自定义key大小比较规则
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
        //对红黑树进行遍历搜索
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);//如果该节点存在，替换并返回
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        //不存在则新建结点插入
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        //红黑树平衡调整
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
    
```

### 查找

```java
//以log(n)的复杂度进行get
final Entry<K,V> getEntry(Object key) {
        // Offload comparator-based version for sake of performance
        if (comparator != null)
            return getEntryUsingComparator(key);
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        Entry<K,V> p = root;
        //遍历红黑树找到相同的元素返回
        while (p != null) {
            int cmp = k.compareTo(p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
        return null;
    }
    
```

### success后继

```java
/**
 * 宏观上讲，TreeMap通过对红黑树进行中序遍历保证其迭代输出是有序的。迭代器
 * 的next方法会调用successor取得后继。
 */
    static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
        if (t == null)
            return null;
        //有右子树的结点，后继结点是右子树的“最左结点”，
        //因为最左子树就是右子树的最小结点
        else if (t.right != null) {
            Entry<K,V> p = t.right;
            while (p.left != null)
                p = p.left;
            return p;
        } else {
        //若右子树为空，寻找当前结点所在左子树的第一个祖先结点
            Entry<K,V> p = t.parent;
            Entry<K,V> ch = t;
            //保证左子树，即父结点的右子树不指向它
            while (p != null && ch == p.right) {
                ch = p;
                p = p.parent;
            }
            return p;
        }
    }
```