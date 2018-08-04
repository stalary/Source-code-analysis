* [介绍](#介绍)
* [常量](#常量)
* [add](#add)
* [remove](#remove)
* [element](#element)
* [offer](#offer)
* [poll](#poll)
* [peek](#peek)
* [push](#push)
* [pop](#pop)


### 介绍
  - 基于双向链表实现
  - 线程不安全
  - 插入删除效率较高，但不支持随机查找

### 常量
```java
// 元素数量
transient int size = 0;

// 头节点
transient Node<E> first;

// 尾节点
transient Node<E> last;
```

### add
```java
    // 插入尾部，并返回true
    // 与addLast方法等价
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
```

```java
    void linkLast(E e) {
        // 获取尾部元素
        final Node<E> l = last;
        // 实例化一个新的节点，前一个节点为l，当前节点为传入的添加节点，下一个节点为null
        final Node<E> newNode = new Node<>(l, e, null);
        // 将尾部节点进行更新(添加上新加入的节点)
        last = newNode;
        // 当尾部节点为空时，头节点即为新添加的节点
        if (l == null)
            first = newNode;
        // 否则，尾部节点的下一个为新添加的节点
        else
            l.next = newNode;
        // 元素数量+1
        size++;
        // 修改次数+1
        modCount++;
    }
```

### remove
```java
    // 删除指定下标的元素
    public E remove(int index) {
        // 检测下标是否越界
        checkElementIndex(index);
        // 调用unlink删除元素
        return unlink(node(index));
    }
```

```java
    E unlink(Node<E> x) {
        // assert x != null;
        // 获取当前元素
        final E element = x.item;
        // 获取下一个节点
        final Node<E> next = x.next;
        // 获取上一个节点
        final Node<E> prev = x.prev;
        // 当上一个节点为null时，将当前节点的下一个节点设置为头节点
        if (prev == null) {
            first = next;
        // 不为null时，将上一个节点的下一个节点设置为当前节点的下一个节点(跳过当前元素)，然后将当前节点的前一个节点设置为null，断开连接
        } else {
            prev.next = next;
            x.prev = null;
        }
        // 当下一个节点为null时，将尾节点设置为上一个节点
        if (next == null) {
            last = prev;
        // 否则将下一个节点的前一个节点设置为前一个节点，并且将当前节点的下一个节点设置为null，断开连接
        } else {
            next.prev = prev;
            x.next = null;
        }
        // 设置当前元素为null
        x.item = null;
        // 元素数量-1
        size--;
        // 修改次数+1
        modCount++;
        // 返回当前元素
        return element;
    }
```

```java
    // 删除头节点
    public E remove() {
        // 直接调用removeFirst，删除头节点
        return removeFirst();
    }
```

```java
    public E removeFirst() {
        // 获取头节点
        final Node<E> f = first;
        // 头节点为null时直接抛出
        if (f == null)
            throw new NoSuchElementException();
        // 调用删除头节点的方法
        return unlinkFirst(f);
    }
```

```java
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        // 获取当前元素
        final E element = f.item;
        // 获取下一个节点
        final Node<E> next = f.next;
        // 将当前元素设置为null
        f.item = null;
        // 将下一个节点设置为null
        f.next = null; // help GC
        // 因为为删除头节点，所以将头节点设置为下一个节点(跳过当前头节点)
        first = next;
        // 当下一个节点为null时，尾节点设置为null
        if (next == null)
            last = null;
        // 否则下一个节点的前一个节点设置为null，断开与之前头节点的连接
        else
            next.prev = null;
        // 元素数量-1
        size--;
        // 操作次数+1
        modCount++;
        // 返回删除的元素
        return element;
    }
```

### element
```java
    // 获取头节点但不删除，链表为null抛出异常
    public E element() {
        // 直接调用getFirst获取头节点返回
        return getFirst();
    }
```

```java
    public E getFirst() {
        // 获取头节点
        final Node<E> f = first;
        // 当头节点为null时抛出异常
        if (f == null)
            throw new NoSuchElementException();
        // 否则返回头节点的值
        return f.item;
    }
```

### offer
> 直接调用add方法

### poll
```java
    // 获取头节点并删除
    public E poll() {
        // 获取头节点
        final Node<E> f = first;
        // 当为null时直接返回null，负责调用unlinkFirst，之前已经写注释了，这里不再解释
        return (f == null) ? null : unlinkFirst(f);
    }
```

### peek 
```java
    // 获取头节点，但不删除，链表为null则返回null
    public E peek() {
        // 获取头节点
        final Node<E> f = first;
        // 头节点为null直接返回null，否则返回元素
        return (f == null) ? null : f.item;
    }
```

### push
```java
    // 压入元素
    public void push(E e) {
        // 直接调用addFirst方法,addFirst调用了linkFirst
        addFirst(e);
    }
```

```java
private void linkFirst(E e) {
        // 获取头节点
        final Node<E> f = first;
        // 实例化一个新的节点，上一个节点为null，当前节点为传入元素，下个节点为头节点
        final Node<E> newNode = new Node<>(null, e, f);
        // 将新的节点赋值给头节点
        first = newNode;
        // 当头节点为空时，尾节点也直接设置为新节点
        if (f == null)
            last = newNode;
        // 否则头节点的前一个节点为新节点
        else
            f.prev = newNode;
        // 元素数量+1
        size++;
        // 修改次数+1
        modCount++;
    }
```

### pop
```java
    // 弹出元素
    public E pop() {
        // 直接调用removeFirst方法，removeFirst进行判空，空抛出异常，然后调用unlinkFirst，上面已经注释过，不再解释
        return removeFirst();
    }
```











