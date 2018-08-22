- [介绍](#%E4%BB%8B%E7%BB%8D)
- [常量&变量](#%E5%B8%B8%E9%87%8F%E5%8F%98%E9%87%8F)
- [put](#put)
- [take](#take)
- [offer](#offer)
- [poll](#poll)
- [peek](#peek)
- [size](#size)
### 介绍
- 基于数组实现的阻塞队列
- 线程安全，基于ReentrantLock实现
- 需要设置容量
  
### 常量&变量
```java
    // 队列中的元素
    final Object[] items;

    // 下一次进行take,poll,peek,remove的下标
    int takeIndex;

    // 下一次进行put,offer,add的下标
    int putIndex;

    // 数组中元素数量
    int count;

    /*
     * Concurrency control uses the classic two-condition algorithm
     * found in any textbook.
     */

    // 所有操作
    final ReentrantLock lock;

    // 用于take时的阻塞
    private final Condition notEmpty;

    // 用于put时的阻塞
    private final Condition notFull;

    // 共享当前活跃的迭代器状态
    transient Itrs itrs = null;
```

### put
```java
    // 阻塞式入队
    public void put(E e) throws InterruptedException {
        // 判断传入的值是否合法
        checkNotNull(e);
        // 获取公共锁
        final ReentrantLock lock = this.lock;
        // 申请可中断的锁
        lock.lockInterruptibly();
        try {
            // 当容量满时，进入等待
            while (count == items.length)
                notFull.await();
            // 当容量不满时，进行入队操作
            enqueue(e);
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```

```java
    private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        // 将当前元素赋值
        items[putIndex] = x;
        // 移动下标，当容量满时，下一次放入的下标设置为0
        if (++putIndex == items.length)
            putIndex = 0;
        // 元素数量+1
        count++;
        // 表示当前队列已经不为空
        notEmpty.signal();
    }
```

### take
```java
    // 阻塞式出队
    public E take() throws InterruptedException {
        // 获取公共锁
        final ReentrantLock lock = this.lock;
        // 申请可中断锁
        lock.lockInterruptibly();
        try {
            // 当队列为空时等待
            while (count == 0)
                notEmpty.await();
            // 队列不为空时进行出队操作
            return dequeue();
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```

```java
    private E dequeue() {
        // assert lock.getHoldCount() == 1;
        // assert items[takeIndex] != null;
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        // 获取下标对应的元素
        E x = (E) items[takeIndex];
        // 设置为null，直接回收
        items[takeIndex] = null;
        // 移动下标，当队列为空时，下一次获取的下标为0
        if (++takeIndex == items.length)
            takeIndex = 0;
        // 数量-1
        count--;
        // 当迭代器不是null时，移动迭代器
        if (itrs != null)
            itrs.elementDequeued();
        // 表示当前队列不是满的
        notFull.signal();
        // 返回出队元素
        return x;
    }
```

### offer
```java
    // 入队
    public boolean offer(E e) {
        // 判断传入的值是否合法
        checkNotNull(e);
        // 获取公共锁
        final ReentrantLock lock = this.lock;
        // 加锁
        lock.lock();
        try {
            // 当容量已满时，直接返回false
            if (count == items.length)
                return false;
            else {
                // 否则进行入队操作，并且返回true
                enqueue(e);
                return true;
            }
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```

### poll
```java
    // 出队
    public E poll() {
        // 获取公共锁
        final ReentrantLock lock = this.lock;
        // 加锁
        lock.lock();
        try {
            // 当数量为0时直接返回null，否则进行出队操作
            return (count == 0) ? null : dequeue();
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```

### peek
```java
    // 查看队头元素
    public E peek() {
        // 获取锁
        final ReentrantLock lock = this.lock;
        加锁
        lock.lock();
        try {
            // 定位元素，直接返回数组对应下标
            return itemAt(takeIndex); // null when queue is empty
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```

### size
```java
    // 获取当前队列元素数量
    public int size() {
        // 获取公共锁
        final ReentrantLock lock = this.lock;
        // 加锁
        lock.lock();
        try {
            // 直接返回数量
            return count;
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```