### 介绍

- TreeSet是个有序的Set，继承了 AbstractSet ，实现了NavigableSet, Cloneable, Serializable接口。
- 默认自然排序，也可指定Comparator。
- 提供了一系列导航方法，查找和指定目标的最匹配项。
- add、remove、contains这些基础操作提供O(logn)的效率。
- 非同步，线程不安全。
- size、isEmpty、contains、add、remove、clear都依托内部的NavigableMap<E,Object>实例实现

### const

```java

//依托内部的NavigableMap<E,Object>实例实现
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
