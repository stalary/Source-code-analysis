- [介绍](#介绍)
- [常量](#常量)
- [构造器和容量相关](#构造器和容量相关)
- [size](#size)
- [add&offer](#add&offer)
- [poll&remove](#poll&remove)
- [get&element&peek](#get&element&peek)
- [clear](#clear)
- [Stack](#Stack)

### 介绍
- 线程不安全。队列里不许有空元素
- 动态数组实现的双向循环队列
- 继承AbstractCollection，实现Deque, Cloneable, Serializable接口
- 双端可操作，因此可单端操作作为栈，双端操作作为队列。
- 用作栈时比Stack快，用作队列时比LinkedList快
- 主要的插入移除方法为addFirst、addLast、pollFirst、pollLast，实现了Deque接口的方法，其他方法都是根据这些派生的。
- 循环队列具体实现细节：
  1. head指向第一个元素
  2. tail指向最后一个元素的后一个位置
  3. head == tail为空，add操作时head == nail队满自动扩容
  4. head < tail，下标区间[head,tail - 1]
  5. head > tail，下标区间[head,elements.length()-1] + [0, tail-1]


### 常量

```java

/*
数组里的元素有序，队列的容量就是这个数组的大小了，
应该是2的幂。数组不会放满，除了我们add的时候调用
doubleCapacity,防止首尾指针相遇的时候。
保证数组内非队列元素即null。
*/
transient Object[] elements; //包访问权限

//最小的初始化容量。必须是2的幂，这里是8.
private static final int MIN_INITIAL_CAPACITY = 8;

//首尾指针
transient int head;
transient int tail;
```

### 构造器和容量相关
```java
//默认 pow(2,4)
public ArrayDeque() {
    elements = new Object[16];
}

//分配最小的n = 2 ^ k，且满足n > numElements
private void allocateElements(int numElements) {
    elements = new Object[calculateSize(numElements)];
}

public ArrayDeque(int numElements) {
    allocateElements(numElements);
}


public ArrayDeque(Collection<? extends E> c) {
    allocateElements(c.size());
    addAll(c);
}

//实际计算空间的算法
private static int calculateSize(int numElements) {
    //不能小于这个最小容量
    int initialCapacity = MIN_INITIAL_CAPACITY;
   //求n的算法，二进制运算
    if (numElements >= initialCapacity) {
        //假设第一位是1，后面无所谓。1
        initialCapacity = numElements;
        initialCapacity |= (initialCapacity >>>  1);
        initialCapacity |= (initialCapacity >>>  2);
        initialCapacity |= (initialCapacity >>>  4);
        initialCapacity |= (initialCapacity >>>  8);
        initialCapacity |= (initialCapacity >>> 16);
        //经过一连串的运算，低位全部变成1，加1之后多一位1，后面全为0
        initialCapacity++;

        //超过了int范围就无符号右移一位，int首位为1则为负数
        if (initialCapacity < 0)   
            initialCapacity >>>= 1; //对于带符号的int来说就是2 ^ 30最大了
    }
    return initialCapacity;
}
//双倍扩容
private void doubleCapacity() {
    assert head == tail; //断言只有在head==tail即队满才能用
    int p = head;
    int n = elements.length;
    int r = n - p; //head右边的元素个数
    int newCapacity = n << 1; //双倍容量
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    System.arraycopy(elements, p, a, 0, r);
    System.arraycopy(elements, 0, a, r, p);
    elements = a;
    head = 0; //重新放到头上
    tail = n;
}
```

### size
tail - head可能为负值，这样需要通过&(element.length - 1)修正一下。
原理是这样的:

假设tail = 3，head = 6，tail - head = -3，element.length = 8

转化为二进制 

11111101 & 00000111 = 0000101 = 5 = element.length - head + tail

因为element.length是2的幂，所以element.length-1，都是这样的形式00001111，可做掩码使用，截取element.length大小内的最后几位。这里简化修正是规定容量必须为2的幂的一个好处，当然2的幂的容量也可以提高内存分配的效率。

```java
public int size() {
    return (tail - head) & (elements.length - 1);
}

```

### add&offer

```java

//插到队首
public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
    //修正head-1
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)
        doubleCapacity();
}
//插到队尾
public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[tail] = e;
    //判满
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)
        doubleCapacity();
}

//***下面依托addX***

public boolean add(E e) {
    addLast(e);
    return true;
}

public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }

public boolean offerLast(E e) {
    addLast(e);
    return true;
}

//入队
public boolean offer(E e) {
    return offerLast(e);
}
```

### poll&remove
```java
// 移除队首
public E pollFirst() {
    int h = head;
    @SuppressWarnings("unchecked")
    E result = (E) elements[h];
    //队空为null
    if (result == null)
        return null;
    elements[h] = null; //设空
    head = (h + 1) & (elements.length - 1);
    return result;
}
//移除队尾
public E pollLast() {
    int t = (tail - 1) & (elements.length - 1);
    @SuppressWarnings("unchecked")
    E result = (E) elements[t];
    if (result == null)
        return null;
    elements[t] = null;
    tail = t;
    return result;
}

//***下面依托pollX***

//出队
public E poll() {
    return pollFirst();
}

public E removeFirst() {
    E x = pollFirst();
    if (x == null)
        throw new NoSuchElementException();
    return x;
}

public E removeLast() {
    E x = pollLast();
    if (x == null)
        throw new NoSuchElementException();
    return x;
}
public E remove() {
    return removeFirst();
}

```

### get&element&peek

```java
//获取队首
public E getFirst() {
    @SuppressWarnings("unchecked")
    E result = (E) elements[head];
    if (result == null)
        throw new NoSuchElementException();
    return result;
}

//获取队尾
public E getLast() {
    @SuppressWarnings("unchecked")
    E result = (E) elements[(tail - 1) & (elements.length - 1)];
    if (result == null)
        throw new NoSuchElementException();
    return result;
}
//队首
public E element() {
    return getFirst();
}

//***下面不抛异常***
public E peekFirst() {
    //空则返回null
    return (E) elements[head];
}

@SuppressWarnings("unchecked")
public E peekLast() {
    return (E) elements[(tail - 1) & (elements.length - 1)];
}

```

### clear

```java
//清空，不在队列范围内的都设为null
public void clear() {
    int h = head;
    int t = tail;
    if (h != t) { // clear all cells
        head = tail = 0;
        int i = h;
        int mask = elements.length - 1;
        do {
            elements[i] = null;
            i = (i + 1) & mask;
        } while (i != t);
    }
}


```
### Stack

```
//查看栈顶
public E peek() {
    return peekFirst();
}

//压入栈顶
public void push(E e) {
    addFirst(e);
}

//弹出栈顶
public E pop() {
    return removeFirst();
}


```