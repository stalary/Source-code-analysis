### 介绍
- 支持一个线程对锁的重复获取
- 有公平和非公平两种模式

### Sync
```java
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        // 非公平方式获取锁
        final boolean nonfairTryAcquire(int acquires) {
            // 获取当前线程
            final Thread current = Thread.currentThread();
            // 获取状态
            int c = getState();
            // 当没有锁时
            if (c == 0) {
                // cas获取锁
                if (compareAndSetState(0, acquires)) {
                    // 获取锁成功以后，将锁的拥有者设置为当前线程
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            // 存在锁时，如果锁的拥有者为当前线程即重入
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                // 锁的数量超过最大值，抛出error
                if (nextc < 0) 
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        // 释放锁
        protected final boolean tryRelease(int releases) {
            // 获取释放锁后的数量
            int c = getState() - releases;
            // 如果当前线程不是锁拥有者，抛出异常
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            // 当已经没有锁时，将锁的拥有者设置为null，否则减少数量
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
    }
```

### NonfairSync
```java
    // 非公平的同步器
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        // 获取锁
        final void lock() {
            // 使用cas设置锁
            if (compareAndSetState(0, 1))
                // 设置成功后，将当前线程设置为锁的持有者
                setExclusiveOwnerThread(Thread.currentThread());
            else
                // 否则调用AQS的acquire方法
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            // 直接调用Sync的非公平获取锁
            return nonfairTryAcquire(acquires);
        }
    }
```

### FairSync
```java
    // 公平的同步器
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        // 直接调用AQS的acquire方法
        final void lock() {
            acquire(1);
        }

        // 使用公平的方式尝试获取锁
        protected final boolean tryAcquire(int acquires) {
            // 获取当前线程
            final Thread current = Thread.currentThread();
            // 获取当前状态
            int c = getState();
            // 当没有锁时，进行获取
            if (c == 0) {
                // 判断是否没有等待者或者为头节点，然后进行cas设置锁
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            // 当前线程为锁的持有者时进行设置
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                // 锁超过最大数量时抛出error
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

