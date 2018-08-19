- [介绍](#%E4%BB%8B%E7%BB%8D)
- [Sync](#sync)
- [await](#await)
- [countDown](#countdown)
- [getCount](#getcount)
### 介绍
- 协调多个线程之间的同步
- 可用于控制多个线程的并行

### Sync
```java
    // 同步控制器，使用AQS来控制状态
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        // 设置AQS状态，写入线程数量
        Sync(int count) {
            setState(count);
        }
        // 获取当前数量
        int getCount() {
            return getState();
        }
        // 尝试获取共享锁，当无锁时返回1，否则返回-1
        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }
        // 尝试释放锁，当已经无锁时返回false，否则使用cas将状态-1，当-1后无锁返回true
        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }
```

### await
```java
    // 等待，当接收到interrupted时抛出异常
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```

### countDown
```java
    // 减少latch的数量，减少到0时，通知所有线程运行
    public void countDown() {
        sync.releaseShared(1);
    }
```

### getCount
```java
    // 获取当前等待中的线程数量
    public long getCount() {
        return sync.getCount();
    }
```