### 介绍
- 基于FIFO队列
- 构建锁的基础框架
- 使用int来表示状态的同步器

### 常量&变量
```java
// 保存线程引用和线程状态的容器
static final class Node {
        // 共享模式
        static final Node SHARED = new Node();
        
        // 排他模式
        static final Node EXCLUSIVE = null;

        // 表示当前线程被取消
        static final int CANCELLED =  1;
        
        // 表示当前节点的后继节点包含的线程需要运行
        static final int SIGNAL    = -1;
        
        // 表示当前节点在等待condition，即在condition队列中
        static final int CONDITION = -2;
        
        // 表示当前场景下后续的acquireShared能够得以执行
        static final int PROPAGATE = -3;

        // 表示节点的状态
        volatile int waitStatus;

        // 前驱节点，当前节点被取消时，需要前驱节点和后继节点完成链接
        volatile Node prev;

        // 后继节点
        volatile Node next;

        // 入队列时的线程
        volatile Thread thread;

        // 存储在condition队列中的后继节点
        Node nextWaiter;
}

// 等待队列的头节点，仅通过setHead修改，当head存在时，等待状态必然不为CANCELLED
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
        // 尝试一次快速入队
        Node pred = tail;
        if (pred != null) {
            // 当前node的前置节点为原尾节点
            node.prev = pred;
            // cas尝试设置尾节点
            if (compareAndSetTail(pred, node)) {
                // 原尾节点的后置节点为当前节点
                pred.next = node;
                return node;
            }
        }
        // 快速入队失败，使用enq入队
        enq(node);
        return node;
    }
```

### enq
```java
    // 插入队列
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
                // 新加入Node的前置为尾节点
                node.prev = t;
                // cas替换尾节点
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
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

### unparkSuccessor
```java
    // 唤醒后置节点
    private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```