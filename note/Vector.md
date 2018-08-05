- [介绍](#介绍)
- [添加](#添加)
- [删除](#删除)
- [查找](#查找)
- [修改](#修改)
- [其他](#其他)

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

public synchronized boolean removeElement(Object obj) {
    modCount++;
    // 获取元素obj的索引，根据indexOf的源码，若Vector中存在多个obj，将返回遍历Vector得到的第一个obj的索引
    int i = indexOf(obj);
    // 若元素obj存在
    if (i >= 0) {
    
        removeElementAt(i);
        return true;
    }
    return false;
}