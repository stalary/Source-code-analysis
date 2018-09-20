- [介绍](#%E4%BB%8B%E7%BB%8D)
- [常量&变量](#%E5%B8%B8%E9%87%8F%E5%8F%98%E9%87%8F)
- [修改状态](#%E4%BF%AE%E6%94%B9%E7%8A%B6%E6%80%81)
- [acquire](#acquire)
- [addWaiter](#addwaiter)
- [enq](#enq)
- [acquireQueued](#acquirequeued)
- [setHead](#sethead)
- [cancelAcquire](#cancelacquire)
- [unparkSuccessor](#unparksuccessor)
- [doReleaseShared](#doreleaseshared)
### 介绍
- AQS的实现依赖内部的同步队列（FIFO双向队列）
- 构建锁的基础框架
- 使用数字来表示持有锁的数量
- 如果当前线程获取同步状态失败，AQS会将该线程以及等待状态等信息构造成一个Node，将其加入同步队列的尾部，同时阻塞当前线程，当同步状态释放时，唤醒队列的头节点。

### 常量&变量
```java
// 保存线程引用和线程状态的容器
static final class Node {
        // 共享模式
        static final Node SHARED = new Node();
        
        // 排他模式
        static final Node EXCLUSIVE = null;

        // 表示当前线程被取消，处于结束状态
        static final int CANCELLED =  1;
        
        // 处于唤醒状态，前继节点释放锁或者被取消就会被唤醒
        static final int SIGNAL = -1;
        
        // 处于等待队列中，调用singal后移到同步队列中
        static final int CONDITION = -2;
        
        // 可运行状态
        static final int PROPAGATE = -3;

        // 表示节点的状态
        volatile int waitStatus;

        // 前驱节点，当前节点被取消时，需要前驱节点和后继节点完成链接
        volatile Node prev;

        // 后继节点
        volatile Node next;

        // 入队列时的线程
        volatile Thread thread;

        // 存储在condition队列中的后继节点
        Node nextWaiter;
}

// 等待队列的头节点，仅通过setHead修改，当head存在时，等待状态必然不为CANCELLED
private transient volatile Node head;

// 等待队列的尾节点，仅通过enq修改
private transient volatile Node tail;

// 同步状态
private volatile int state;

// 自旋超时时间
static final long spinForTimeoutThreshold = 1000L;
```

### 修改状态
```java
    // 直接修改
    protected final void setState(int newState) {
        state = newState;
    }

    // 使用cas原子的更新状态，防止多个线程同时修改
    protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```

### acquire
```java
    // 以排他的方式获取锁
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

### addWaiter
```java
    // 以指定模式创建node并加入到队列当中
    private Node addWaiter(Node mode) {
        // 创建指定模式的node
        Node node = new Node(Thread.currentThread(), mode);
        // 尝试一次快速入队
        Node pred = tail;
        if (pred != null) {
            // 当前node的前置节点为原尾节点
            node.prev = pred;
            // cas尝试设置尾节点
            if (compareAndSetTail(pred, node)) {
                // 原尾节点的后置节点为当前节点
                pred.next = node;
                return node;
            }
        }
        // 快速入队失败，使用enq入队
        enq(node);
        return node;
    }
```

### enq
```java
    // 插入队列
    private Node enq(final Node node) {
        // 自旋
        for (;;) {
            Node t = tail;
            // 需要初始化
            if (t == null) { 
                // 使用cas设置头节点，初始化
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                // 新加入Node的前置为尾节点
                node.prev = t;
                // cas替换尾节点，只替换了tail的地址，局部变量t不会替换
                if (compareAndSetTail(t, node)) {
                    // 将新加入的节点加到末尾
                    t.next = node;
                    // 返回前置节点，调用方需要通过前置节点来唤醒后置节点
                    return t;
                }
            }
        }
    }
```

### acquireQueued
```java
    // 请求入队
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            // 自选
            for (;;) {
                // 获取当前节点的前置节点
                final Node p = node.predecessor();
                // 当前置节点为头节点并且尝试获取锁成功时，唤醒下一个节点
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 需要等待
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            // 自旋失败时，取消获取锁
            if (failed)
                cancelAcquire(node);
        }
    }
```

### setHead
```java
    // 设置头节点
    private void setHead(Node node) {
        head = node;
        // 设置为null，gc回收
        node.thread = null;
        node.prev = null;
    }
```

### cancelAcquire
```java
    // 取消获取锁
    private void cancelAcquire(Node node) {
        // 节点null时直接返回
        if (node == null)
            return;
        node.thread = null;
        // 跳过被取消的前置节点
        Node pred = node.prev;
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;
        Node predNext = pred.next;
        // 当前节点状态设置为取消
        node.waitStatus = Node.CANCELLED;
        // 如果当前节点为尾节点，更新为前置节点，然后设置下一个节点
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {
            int ws;
            // 如果node既不是tail，又不是head的后继节点，将前置节点状态设置为SIGNAL，然后将node从队列中删除
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                if (next != null && next.waitStatus <= 0)
                    compareAndSetNext(pred, predNext, next);
            } else {
                // 当node时head的后继节点时，直接唤醒
                unparkSuccessor(node);
            }
            node.next = node;
        }
    }
```

### unparkSuccessor
```java
    // 唤醒后置节点
    private void unparkSuccessor(Node node) {
        // 获取节点状态
        int ws = node.waitStatus;
        if (ws < 0)
            // 小于0时，cas设置为0
            compareAndSetWaitStatus(node, ws, 0);

        // unpark后继节点，一般是下一个节点，所以获取当前节点的下一个节点
        Node s = node.next;
        // 下一个节点为空，并且等待状态>0时
        if (s == null || s.waitStatus > 0) {
            s = null;
            // 遍历节点，找到一个需要唤醒的节点
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        // 下一个节点不为空的时候，调用unpark
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

###  doReleaseShared
```java
    private void doReleaseShared() {
        // 自旋cas释放共享锁
        for (;;) {
            // 获取头节点
            Node h = head;
            if (h != null && h != tail) {
                // 获取当前等待状态
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    // 需要被唤醒时使用vas进行唤醒，失败后进入自旋
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;           
                    // unpark后继节点
                    unparkSuccessor(h);
                }
                // 当不需要等待时，设置为可执行
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;          
            }
            if (h == head)                   
                // 修改成功后结束循环
                break;
        }
    }
```
