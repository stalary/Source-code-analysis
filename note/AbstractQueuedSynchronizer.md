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
```