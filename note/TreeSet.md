- [介绍](#介绍)
- [const&field](#constfield)
- [constructor](#constructor)
- [addAll](#addall)
- [subSet、headSet、tailSet](#subsetheadsettailset)
- [导航(索引)方法](#导航索引方法)

### 介绍

- TreeSet是个有序的Set，继承了 AbstractSet ，实现了NavigableSet, Cloneable, Serializable接口。
- 默认自然排序，也可指定Comparator。
- 提供了一系列导航方法，查找和指定目标的最匹配项。
- add、remove、contains这些基础操作提供O(logn)的效率。
- 非同步，线程不安全。
- size、isEmpty、contains、add、remove、clear都依托内部的NavigableMap<E,Object>实例实现

### const&field

```java

//依托内部的NavigableMap<E,Object>，实际上是TreeMap做内部容器
private transient NavigableMap<E,Object> m;

//填补Map中的value
private static final Object PRESENT = new Object();


```


### constructor

```java

// 默认构造
TreeSet()

// 根据collection构造
TreeSet(Collection<? extends E> collection)

// 指定Comparator的构造
TreeSet(Comparator<? super E> comparator)

// 根据SortedSet构造
TreeSet(SortedSet<E> set)

//根据NavigableMap<E,Object>构造
TreeSet(NavigableMap<E,Object> m)

```

### addAll

```java


//加入集合c中全部元素
public  boolean addAll(Collection<? extends E> c) {
    // Use linear-time version if applicable
    if (m.size()==0 && c.size() > 0 &&
        c instanceof SortedSet &&
        m instanceof TreeMap) {
        //类型转换
        SortedSet<? extends E> set = (SortedSet<? extends E>) c; 
        TreeMap<E,Object> map = (TreeMap<E, Object>) m;
        Comparator<? super E> cc = (Comparator<? super E>) set.comparator();
        Comparator<? super E> mc = map.comparator();
        
        //如果cc和mc两个Comparator相等
        if (cc==mc || (cc != null && cc.equals(mc))) {
        //把Collection中所有元素添加成TreeMap集合的key
            map.addAllForTreeSet(set, PRESENT);
            return true;
        }
    }
    

```

### subSet、headSet、tailSet

```java

//***通过TreeMap的subMap()实现,inclusive为是否包含边界***

// 返回子Set，从fromElement到toElement。
public NavigableSet<E> subSet(E fromElement, boolean fromInclusive,
                              E toElement,   boolean toInclusive) {
    return new TreeSet<E>(m.subMap(fromElement, fromInclusive,
                                   toElement,   toInclusive));
}

//从头部到toElement，inclusive为是否包含toElement
public NavigableSet<E> headSet(E toElement, boolean inclusive) {
    return new TreeSet<E>(m.headMap(toElement, inclusive));
}

//从fromElement到结尾。
public NavigableSet<E> tailSet(E fromElement, boolean inclusive) {
    return new TreeSet<E>(m.tailMap(fromElement, inclusive));
}
    
```

### 导航(索引)方法

```java

//小于e的最大元素
public E lower(E e) {
    return m.lowerKey(e);
}

//小于/等于e的最大元素
public E floor(E e) {
    return m.floorKey(e);
}

//大于/等于e的最小元素
public E ceiling(E e) {
    return m.ceilingKey(e);
}

//中大于e的最小元素
public E higher(E e) {
    return m.higherKey(e);
}

```

