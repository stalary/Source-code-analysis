- [介绍](#介绍)
- [const&field](#constfield)
- [构造](#构造)
- [size和isEmpty和clear](#sizeisemptyclear)
- [contains](#contains)
- [add](#add)
- [remove](#remove)
- [clone](#clone)

### 介绍
继承了AbstractSet，实现了Set, Cloneable,Serializable。内部是依赖HashMap实现的，所以HashSet的结构比较简单。不保证元素的迭代顺序一直不变，允许有一个null。非同步的，可以通过

> Set s = Collections.synchronizedSet(new HashSet());

进行包装，实现对外同步。

### const&field

```java

//依赖的HashMap实例
private transient HashMap<E,Object> map;

//一个static final的空对象，用于填补hashmap的key对应的value
private static final Object PRESENT = new Object();

```

### 构造

```java

//默认构造，构造一个空set，内部的HashMap实例容量为初始的16，负载因子0.75
public HashSet() {
    map = new HashMap<>();
}

//用足够的容量和默认的负载因子构造，包含集合里的元素
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}

//用给定的容量和负载因子构造
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}


//用给定的容量和默认的负载因子构造
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

```

### size&isEmpty&clear

```java

//***都是对内部HashMap实例的操作***

public int size() {
    return map.size();
}

public boolean isEmpty() {
    return map.isEmpty();
}

public void clear() {
    map.clear();
}
```

### contains

```java

//调用HashMap#containsKey查找，因为set内元素被当做key存储在hashmap中
public boolean contains(Object o) {
    return map.containsKey(o);
}


```

### add

```java
//将e当做key插入，因为key是不可重复的，也就保证了e在set中唯一
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

```


### remove

```java

//在hashmap中根据key删除一个entry,删除成功就会返回之前置入的PRESENT
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}

```

### clone

```java

//对内部的HashMap实例做一个深拷贝
public Object clone() {
    try {
        HashSet<E> newSet = (HashSet<E>) super.clone();
        newSet.map = (HashMap<E, Object>) map.clone();
        return newSet;
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }

```