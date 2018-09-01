- [介绍](#%E4%BB%8B%E7%BB%8D)
- [Sync](#sync)
- [NonfairSync](#nonfairsync)
- [FairSync](#fairsync)
- [acquire](#acquire)
- [release](#release)
### 介绍
- 设置同一时间被访问的线程数量，可以实现流量控制
- 可用于限流

### Sync
```java
    // 同步控制器，使用AQS来控制状态
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 1192457210091910933L;
        // 设置信号量的数量
        Sync(int permits) {
            setState(permits);
        }
        // 获取当前信号量数量
        final int getPermits() {
            return getState();
        }
        // 非公平的获取共享锁
        final int nonfairTryAcquireShared(int acquires) {
            // 自旋cas操作
            for (;;) {
                // 获取当前信号量
                int available = getState();
                // 获取当前可使用的信号量的数量
                int remaining = available - acquires;
                // 当前可用数量小于零或者cas设置状态成功时直接返回当前剩余数量
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
        // 释放共享锁
        protected final boolean tryReleaseShared(int releases) {
            // 自旋cas操作
            for (;;) {
                // 获取当前信号量
                int current = getState();
                // 释放后，将信号量归还
                int next = current + releases;
                // 当溢出时，抛出一个溢出error
                if (next < current)
                    throw new Error("Maximum permit count exceeded");
                // 使用cas更新信号量
                if (compareAndSetState(current, next))
                    return true;
            }
        }
        // 减少信号量的数量
        final void reducePermits(int reductions) {
            // 自旋cas操作
            for (;;) {
                // 获取当前信号量的数量
                int current = getState();
                // 减去要减少的数量
                int next = current - reductions;
                // 当溢出时，抛出一个溢出error
                if (next > current) // underflow
                    throw new Error("Permit count underflow");
                // cas修改信号量的数量
                if (compareAndSetState(current, next))
                    return;
            }
        }

        // 将信号量全部使用
        final int drainPermits() {
            // 自旋cas操作
            for (;;) {
                // 获取当前信号量的数量
                int current = getState();
                // 当信号量已经为0或者cas设置为0成功时，返回
                if (current == 0 || compareAndSetState(current, 0))
                    return current;
            }
        }
    }
```

### NonfairSync
```java
    // 非公平同步器
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = -2694183684443567898L;

        NonfairSync(int permits) {
            super(permits);
        }
        // 直接调用同步器中的非公平获取共享锁的方法
        protected int tryAcquireShared(int acquires) {
            return nonfairTryAcquireShared(acquires);
        }
    }
```

### FairSync

```java
    // 公平同步器
    static final class FairSync extends Sync {
        private static final long serialVersionUID = 2014338818796000944L;

        FairSync(int permits) {
            super(permits);
        }
        // 公平的获取共享锁
        protected int tryAcquireShared(int acquires) {
            // 自旋cas操作
            for (;;) {
                // 首先排队，判断是不是队列中的第一个线程
                if (hasQueuedPredecessors())
                    return -1;
                // 获取当前信号量数量
                int available = getState();
                // 减去获取的信号量
                int remaining = available - acquires;
                // 当信号量小于零，无法获取到信号量或者cas修改信号量成功时，返回
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
    }
```

### acquire
```java
    // 获取信号量，每次只能获取一个，使用aqs中可中断的方式
    public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```

```java
    // 获取指定数量的信号量
    public void acquire(int permits) throws InterruptedException {
        // 当数量小于0时，直接抛出异常
        if (permits < 0) throw new IllegalArgumentException();
        sync.acquireSharedInterruptibly(permits);
    }
```

```java
    // 尝试以非公平的方式获取锁，返回获取锁的状态
    public boolean tryAcquire() {
        return sync.nonfairTryAcquireShared(1) >= 0;
    }
```

### release
```java
    // 释放一个信号量
    public void release() {
        sync.releaseShared(1);
    }
```

```java
    // 释放指定数量的信号量
    public void release(int permits) {
        if (permits < 0) throw new IllegalArgumentException();
        sync.releaseShared(permits);
    }
```



