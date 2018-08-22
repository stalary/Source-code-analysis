- [介绍](#%E4%BB%8B%E7%BB%8D)
- [常量&变量](#%E5%B8%B8%E9%87%8F%E5%8F%98%E9%87%8F)
- [get](#get)
- [add](#add)
- [set](#set)
- [remove](#remove)
### 介绍
- 在写时进行复制的线程安全ArrayList
- 适合读多写少的场景
- 保证最终一致性
  
### 常量&变量
```java
    // 控制所有函数的锁
    final transient ReentrantLock lock = new ReentrantLock();

    // 存储数据的数组，设置volatile，保证可见性
    private transient volatile Object[] array;
```

### get
```java
    // 直接无锁访问数组下标获取数据
    public E get(int index) {
        return get(getArray(), index);
    }
```

### add
```java
    // 向list中获取元素
    public boolean add(E e) {
        // 获取公共锁
        final ReentrantLock lock = this.lock;
        // 加锁
        lock.lock();
        try {
            // 获取当前数组
            Object[] elements = getArray();
            // 求出数组长度
            int len = elements.length;
            // 扩容拷贝数组
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            // 将新数组的最后一个元素设置为e
            newElements[len] = e;
            // 赋值给数组
            setArray(newElements);
            return true;
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```

### set
```java
    // 更新指定下标的元素
    public E set(int index, E element) {
        // 获取公共锁
        final ReentrantLock lock = this.lock;
        // 加锁
        lock.lock();
        try {
            // 获取数组
            Object[] elements = getArray();
            // 获取数组插入位置的元素
            E oldValue = get(elements, index);
            // 当值不相等时进行更新
            if (oldValue != element) {
                int len = elements.length;
                // 拷贝数组
                Object[] newElements = Arrays.copyOf(elements, len);
                newElements[index] = element;
                // 赋值给数组
                setArray(newElements);
            } else {
                // Not quite a no-op; ensures volatile write semantics
                // 当值相同时，直接赋值
                setArray(elements);
            }
            // 返回原来的值
            return oldValue;
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```

### remove
```java
    // 删除指定下标的元素
    public E remove(int index) {
        // 获取公共锁
        final ReentrantLock lock = this.lock;
        // 加锁
        lock.lock();
        try {
            // 获取数组
            Object[] elements = getArray();
            // 获取数组长度
            int len = elements.length;
            // 获取当前下标的元素
            E oldValue = get(elements, index);
            // 计算移动的距离
            int numMoved = len - index - 1;
            if (numMoved == 0)
                // 不需要移动时，代表删除的末尾元素
                setArray(Arrays.copyOf(elements, len - 1));
            else {
                // 实例化新的数组
                Object[] newElements = new Object[len - 1];
                // 先拷贝前一部分
                System.arraycopy(elements, 0, newElements, 0, index);
                // 再拷贝后一部分
                System.arraycopy(elements, index + 1, newElements, index,
                                 numMoved);
                // 赋值
                setArray(newElements);
            }
            // 返回删除的值
            return oldValue;
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```