- [介绍](#介绍)
- [const&field](#constfield)
- [构造方法](#构造方法)
- [容量相关](#容量相关)
- [添加](#添加)
- [删除](#删除)
- [查找](#查找)
- [修改](#修改)
- [其他](#其他)

### 介绍

- ArrayList非线程安全。

- ArrayList基于动态数组，是一种线性表。随机访问友好，插入和删除效率低。

- 容量动态调节，有一套扩容和优化空间的机制

- ArrayList继承了AbstractList，实现了List、RandomAccess、Cloneable、Serializable接口。
- Based on Jdk8

### const&field

```java
//元素个数，并不一定是容量
private int size;

//默认初始容量
private static final int DEFAULT_CAPACITY = 10;

//指定初始容量为0时，返回该空数组
private static final Object[] EMPTY_ELEMENTDATA = {};


//不指定初始容量时，返回该空数组。
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/*
存储元素，ArrayList的容量就是这个缓冲区的容量。
当elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA时，
第一次添加元素后，扩容至默认容量10。非私有域便于嵌套类的访问。

另外虽然这里用了transient修饰，但是其实现了readObject和writeObject
  for (int i = 0; i < size; i++)  
            s.writeObject(elementData[i]); 
查看源码可知,实现了序列化
*/
transient Object[] elementData; 

//最大容量，避免在某些虚拟机下可能引起的OutOfMemoryError，减8的原因：数组作为一个对象，需要一定的内存存储对象头信息，对象头信息最大占用内存不可超过8字节。
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

//序列化的VersionUID
private static final long serialVersionUID = 8683452581122892189L;

```

### 构造方法

```java

//默认构造个空list
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}


//根据给定的初始容量构造
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}


//构造一个包含特定元素的list,用iterator依次取出collection中的元素
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        //这里有个bug，c.toArray()可能不会返回Object[] 
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}


```

### 容量相关

```java
//给用户使用，确保容量，指定的容量要大于默认容量
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            ? 0
            : DEFAULT_CAPACITY;
    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}

//给类内部使用，确保容量,用于内部优化,保证空间资源不被浪费, 主要用于add()方法
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity= Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

//实际确保容量的方法
private void ensureExplicitCapacity(int minCapacity) {
    //用于fail-fast机制，用于在并发场景下
    modCount++;
    // 防溢出
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}


//私有扩容方法，确保minCapacity
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    // 扩充当前容量的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 若 newCapacity 依旧小于 minCapacity
    if (newCapacity - minCapacity < 0)  
        newCapacity = minCapacity;
    // 若 newCapacity大于最大存储容量，分配为最大容量
    if (newCapacity - MAX_ARRAY_SIZE > 0)   
        newCapacity = hugeCapacity(minCapacity);
    //
    elementData = Arrays.copyOf(elementData, newCapacity);
}

//私有大容量分配，最大分配Integer.MAX_VALUE,最小分配MAX_ARRAY_SIZE
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // 溢出
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
}



```
### 添加

```java
//加到最后。O(1)
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

//加到指定位置，后面依次后移。O(n)
public void add(int index, E element) {
    rangeCheckForAdd(index);  //检查index
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
//将集合的所有元素添加到末尾
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    //要添加元素的个数
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

//指定位置插入集合元素
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;  
    ensureCapacityInternal(size + numNew);
    int numMoved = size - index;//list中要移动的数量
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}

//前者不检查负值，让jvm抛出ArrayIndexOfBound
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

```

### 删除

```java
//删除指定位置元素并返回 O(n)
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //让gc进行回收
    elementData[--size] = null; 
    return oldValue;
}

//删除给定obj
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}


//私有删除方法，不进行边界检查，不返回被删除元素
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        //native方法，速度比for和clone都"fast"
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null;
}

//删除[fromIndex,toIndex)的元素
protected void removeRange(int fromIndex, int toIndex) {
    modCount++;
    int numMoved = size - toIndex;//要移动的数量
    System.arraycopy(elementData, toIndex, elementData, fromIndex,
            numMoved);

    // 删除后，list 的长度
    int newSize = size - (toIndex-fromIndex);
    //将失效元素置空
    for (int i = newSize; i < size; i++) {
        elementData[i] = null;
    }
    size = newSize;
}

//移除c集合中的元素
public boolean removeAll(Collection<?> c) {
    //判断集合是否为空，否则抛出NPE
    Objects.requireNonNull(c);
    return batchRemove(c, false);
}

//保留c集合中的元素
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}


/*
批量移除。O(n)
第二个参数，如果为true只保留c集合中元素，如果false，移除c集合中的元素
*/
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    //两个指针，r是读取位置，w是写入位置
    int r = 0, w = 0;
    boolean modified = false;
    try {
        //遍历数组，修改元素，这里使用for循环，效率要低于fastremove
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {，
        //此时可能出错，将r之后的拷贝到w之后
        if (r != size) {
            System.arraycopy(elementData, r,
                    elementData, w,
                    size - r);
            w += size - r;
        }
        if (w != size) {
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    //为false则w == size，说明并没有删除
    return modified;
}

//清空list，不释放空间 O(n)
public void clear() {
    modCount++;
    //交给gc吧23333
    for (int i = 0; i < size; i++)
        elementData[i] = null;
    size = 0;
}

```


### 查找

```java
//获取指定位置元素
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}

 
//顺序找，返回首先出现的位置，找不到返-1。O(n)
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}

//逆序找，返回最后出现的位置，找不到返-1。O(n)

public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}

//顺序找实现，根据返回值判断
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}


```

### 修改

```java

//修改指定位置元素
public E set(int index, E element) {
    rangeCheck(index);
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}

```

### 其他

```java
/*
取子list,返回Sublist这个ArrayList的内部类,
这是个坑，注意SubList和其他List实现类的区别
*/
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}

//传入Compartor，用Arrays.sort()实现，主要是LegacyMergeSort和Timsort
public void sort(Comparator<? super E> c) {
    final int expectedModCount = modCount;
    Arrays.sort((E[]) elementData, 0, size, c);
    //并发环境下有可能抛出
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
    modCount++;
}
}

//判空，直接看size就行了
public boolean isEmpty() {
    return size == 0;
}

//克隆，主要拷贝elementData数组
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}

//拷贝到新数组中，释放多余空间
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}


```

