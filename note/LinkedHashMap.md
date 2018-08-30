
### 介绍
- Key和Value都允许空
- 有序
- 非线程安全
- 可用于实现LRU

### 常量&变量
```java
    // 双向链表
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }

    // 头节点，最老的元素
    transient LinkedHashMap.Entry<K,V> head;

    // 尾节点，最新的元素
    transient LinkedHashMap.Entry<K,V> tail;

    //默认是false，则迭代时输出的顺序是插入节点的顺序。若为true，则输出的顺序是按照访问节点的顺序。为true时，可以在这基础之上构建LRU
    final boolean accessOrder;
```

### containsValue
```java
    // 查看是否包含某个元素
    public boolean containsValue(Object value) {
        // 从头到尾遍历双向链表
        for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
            V v = e.value;
            // 直接判断相等或者对象时，使用equals判断值相等时返回true
            if (v == value || (value != null && value.equals(v)))
                return true;
        }
        return false;
    }
```

### get
```java
    public V get(Object key) {
        Node<K,V> e;
        // 调用HashMap方法判断元素是否已经存在
        if ((e = getNode(hash(key), key)) == null)
            return null;
        // 当设置accessOrder时，将当前获取的元素放入
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
```

### afterNodeAccess(LRU的主要实现方法，将元素插入到末尾)
```java
    // 第一种情况，插入节点是头节点
    // a b c d e
    // get(a),将a放到末尾
    // b c d e a
    // 第二种情况，插入节点是尾节点
    // 
    // 将元素移动到最后一个
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        // 当需要排序并且加入节点e不是尾节点时，进入逻辑(为尾节点时，无需操作，已经为最近访问过的元素)
        if (accessOrder && (last = tail) != e) {
            // 首先暂存插入节点的当前节点，上一个节点，下一个节点
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            // 将插入节点的后置节点设为null，代表尾节点
            p.after = null;
            // 如果p的前置节点为null，代表插入节点为头节点，将新的头节点设置为插入节点的下一个节点
            if (b == null)
                head = a;
            else
                // 不为空时，将插入节点的前一个节点的下一个节点设置为插入节点的后一个节点
                b.after = a;
            // 插入节点的后一个节点不为null时，插入节点的后一个节点的前一个节点设置为插入节点的前一个节点
            if (a != null)
                a.before = b;
            else
                // 这一块不知道什么时候会进入？？
                last = b;
            // 尾节点为null，代表链表为空，头节点直接设置为插入节点
            if (last == null)
                head = p;
            else {
                // 头节点的前置节点为尾节点
                p.before = last;
                // 尾节点的后一个节点为插入节点
                last.after = p;
            }
            // 新的尾节点设置为插入节点
            tail = p;
            // 增加修改数量
            ++modCount;
        }
    }
```

### clear
```java
    // 清空map中元素
    public void clear() {
        // 增加modcount，清除数组内元素
        super.clear();
        // 头等于尾等于null，代表链表为空
        head = tail = null;
    }
```

### keySet
```java
    // 获取map的key值Set
    public Set<K> keySet() {
        // 获取keySet
        Set<K> ks = keySet;
        if (ks == null) {
            // 实例化为LinkedKeySet
            ks = new LinkedKeySet();
            keySet = ks;
        }
        return ks;
    }
```

### values
```java
    // 获取map的value值的集合
    public Collection<V> values() {
        // 获取结合元素
        Collection<V> vs = values;
        if (vs == null) {
            // 实例化为LinkedValues
            vs = new LinkedValues();
            values = vs;
        }
        return vs;
    }
```



