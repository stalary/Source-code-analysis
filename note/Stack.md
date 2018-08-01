* [介绍](#介绍)
* [push](#push)
* [pop](#pop)
* [peek](#peek)
* [empty](#empty)
* [search](#search)

### 介绍
Stack类表示存放对象的后进先出（LIFO）栈。它继承了Vector类（这就意味着，Stack也是通过数组实现的，而非链表，且Stack包含Vector中的全部API），并新增了5个方法从而允许在Vector中进行栈操作。这些方法包括入栈和出栈方法，查看栈顶元素的方法，检查栈是否为空的方法，以及在栈中搜索元素并返回该元素距栈顶距离的方法。  
当一个stack对象被实例化时并不包含任何元素。  
Deque接口及其实现类提供了更加完整和一致的栈操作，应优先使用此类。例如：
```java
Deque<Integer> stack = new ArrayDeque<Integer>()
```

### push
push(E item)方法将一个元素推入栈顶，与Vector中的addElement(item)具有相同效果。
```java
public E push(E item) {
    // 入栈实质上是将元素追加到数组的末尾
    addElement(item);

    return item;
}
```

### pop
pop()方法移除栈顶元素并将该元素返回。
```java
public synchronized E pop() {
    E       obj;
    int     len = size();

    // 获取栈顶元素
    obj = peek();
    // 出栈实质上是删除数组末尾的元素
    removeElementAt(len - 1);

    // 返回获取的栈顶元素
    return obj;
}
```

### peek
peek()方法获取栈顶元素，并不执行删除操作
```java
public synchronized E peek() {
    int     len = size();

    if (len == 0)
        throw new EmptyStackException();
    // 返回数组末尾的元素
    return elementAt(len - 1);
}
```

### empty
empty()方法检查栈是否为空
```java
public boolean empty() {
    // 直接调用Vector提供的size()方法，没啥好说的
    return size() == 0;
}
```

### search
如果参数中的对象o是栈中的元素，则此方法返回该元素距栈顶的距离（栈顶元素被认为距栈顶的距离为1）
```java
public synchronized int search(Object o) {
    // 如果栈中含有多个对象o，则返回距栈顶最近的对象o的距离
    int i = lastIndexOf(o);

    if (i >= 0) {
        return size() - i;
     }
     return -1;
}
```