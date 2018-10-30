- [介绍](#%E4%BB%8B%E7%BB%8D)
- [添加](#%E6%B7%BB%E5%8A%A0)
- [删除](#%E5%88%A0%E9%99%A4)
- [查找](#%E6%9F%A5%E6%89%BE)
- [修改](#%E4%BF%AE%E6%94%B9)
- [扩容](#%E5%A2%9E%E5%A4%A7%E5%AE%B9%E9%87%8F)
### 介绍
- Vector是矢量队列，继承于AbstractList，实现了List, RandomAccess, Cloneable和Serializable接口
- Vector继承了AbstractList，实现了List接口，所以它是一个队列，支持相关的添加、删除、修改、遍历等功能
- Vector实现了RandomAccess接口，即提供了随机访问功能。在Vector中，我们可以通过元素的序号快速获取元素对象，这就是快速随机访问。
- Vector实现了Cloneable接口，即实现了clone()方法，可以被克隆
- 和ArrayList不同，Vector中的操作是线程安全的

### 添加
add(E e)方法将元素插入到Vector末尾
```java
public synchronized boolean add(E e) {
    modCount++;
    // 确定插入后的容量没有超过最大容量，否则对Vector进行扩容
    ensureCapacityHelper(elementCount + 1);
    // 将e赋值给elementData[elementCount]，然后将Vector元素数量加1
    elementData[elementCount++] = e;
    return true;
}
```

add(int index, E element)方法将元素插入到指定的index处
```java
public void add(int index, E element) {
    // 直接调用insertElementAt方法
    insertElementAt(element, index);
}
```

```java
public synchronized void insertElementAt(E obj, int index) {
    modCount++;
    if (index > elementCount) {
        throw new ArrayIndexOutOfBoundsException(index
                                                 + " > " + elementCount);
    }
    // 确定插入后的容量没有超过最大容量，否则对Vector进行扩容
    ensureCapacityHelper(elementCount + 1);
    // 使用System.arraycopy将elementData[index]及其之后的元素向后移动一个位置
    System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
    // 将obj赋值给elementData[index]
    elementData[index] = obj;
    // Vector元素数量加1
    elementCount++;
}
```

### 删除
remove(int index)方法删除指定index上的元素
```java
public synchronized E remove(int index) {
    modCount++;
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    // 获取index处原先的值
    E oldValue = elementData(index);

    int numMoved = elementCount - index - 1;
    if (numMoved > 0)
        // 使用System.arraycopy将elementData[index]之后的元素向前移动一个位置
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 将Vector最后一个元素设置为null（以便进行gc），然后将Vector元素数量减1
    elementData[--elementCount] = null;

    // 返回index处原先的值
    return oldValue;
}
```

remove(Object o)删除特定元素o
```java
public boolean remove(Object o) {
    // 直接调用removeElement方法
    return removeElement(o);
}
```

```java
public synchronized boolean removeElement(Object obj) {
    modCount++;
    // 获取元素obj的索引
    // 根据indexOf的源码，若Vector中存在多个obj，将返回遍历Vector得到的第一个obj的索引
    int i = indexOf(obj);
    // 若元素obj存在
    if (i >= 0) {
        // 调用removeElementAt删除指定索引的元素
        // removeElementAt方法与上面的remove(int index)方法基本一致
        removeElementAt(i);
        return true;
    }
    return false;
}
```

### 查找
get方法获取指定index处的元素
```java
public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    // 直接调用elementData方法
    return elementData(index);
}
```

```java
E elementData(int index) {
    // 返回elementData[index]，直截了当
    return (E) elementData[index];
}
```

### 修改
set方法可以修改index处的元素
```java
public synchronized E set(int index, E element) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    // 获取index处原先的值
    E oldValue = elementData(index);

    // 将element赋值给elementData[index]
    elementData[index] = element;
    // 返回index处原先的值
    return oldValue;
}
```

### 扩容
```java
    private void ensureCapacityHelper(int minCapacity) {
        // 当传入容量大于当前容量时，进行扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

```java    
    private void grow(int minCapacity) {
        // 获取当前容量
        int oldCapacity = elementData.length;
        // 当已达到上限时，直接修改为最大容量，否则修改为当前容量+设置的增长容量
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 使用Arrays.copyOf方法将原数组元素复制到容量为newCapacity的新数组中
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```