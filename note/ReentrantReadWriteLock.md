### 介绍
- 读写锁的实现
- 分为公平和非公平模式
- 读写互斥，读共享

### 常量和变量
```java
    // 读锁
    private final ReentrantReadWriteLock.ReadLock readerLock;
    // 写锁
    private final ReentrantReadWriteLock.WriteLock writerLock;
    // 同步器
    final Sync sync;
```

### 同步器
```java
    // 继承了AQS的同步器
    abstract static class Sync extends AbstractQueuedSynchronizer {
        // 读锁占用位
        static final int SHARED_SHIFT   = 16;
        // 增加一个读锁会改变这个值
        static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
        // 锁的最大重入数量
        static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
        // 用于计算写锁的数量
        static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
        // 读锁数量
        static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
        // 写锁数量
        static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
    }
```

### tryRelease
```java
        // 尝试释放锁
        protected final boolean tryRelease(int releases) {
            // 当前未持有写锁，抛出异常
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            // 获取释放后的状态
            int nextc = getState() - releases;
            // 当不存在锁的时候，设置aqs状态为null，即代表释放锁
            boolean free = exclusiveCount(nextc) == 0;
            if (free)
                setExclusiveOwnerThread(null);
            // 修改状态
            setState(nextc);
            return free;
        }
```

### tryReleaseShared
```java
        // 尝试释放共享锁
        protected final boolean tryReleaseShared(int unused) {
            // 获取当前线程
            Thread current = Thread.currentThread();
            // 第一次获得读锁的线程为当前线程
            if (firstReader == current) {
                // 当读锁数量为1时直接释放
                if (firstReaderHoldCount == 1)
                    firstReader = null;
                else
                    // 否则-1
                    firstReaderHoldCount--;
            } else {
                // 获取锁的计数器
                HoldCounter rh = cachedHoldCounter;
                // 当计数器为null或者不是当前线程的计数器时，从ThreadLocal获取
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                // 获取数量
                int count = rh.count;
                if (count <= 1) {
                    // 数量小于等于1时，移除当前线程
                    readHolds.remove();
                    // 当数量小于等于0，代表么有锁，抛出异常
                    if (count <= 0)
                        throw unmatchedUnlockException();
                }
                // 减去锁的数量
                --rh.count;
            }
            // 自旋+cas操作
            for (;;) {
                // 获取当前状态
                int c = getState();
                // 获取下一个状态
                int nextc = c - SHARED_UNIT;
                // 使用cas设置
                if (compareAndSetState(c, nextc))
                    // 锁全部释放返回成功
                    return nextc == 0;
            }
        }
```

### tryAcquire
```java
        // 分为三个步骤
        // 1. 如果读锁或者写锁数量不是0，并且拥有锁的线程不是当前现场，获取失败
        // 2. 如果锁的数量已满，获取失败
        // 3. 否则，如果是可重入，或者可获取锁的获取成功
        protected final boolean tryAcquire(int acquires) {
            // 获取当前线程
            Thread current = Thread.currentThread();
            // 获取当前锁的数量
            int c = getState();
            // 计算写锁的数量
            int w = exclusiveCount(c);
            // 当前存在写锁或者读锁
            if (c != 0) {
                // 不存在写锁或者当前线程不是独占线程时获取失败
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                // 当写锁数量超过最大值时，抛出error
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // 重入
                setState(c + acquires);
                return true;
            }
            // 当线程需要阻塞(占有着写锁)或者cas设置锁失败时返回false
            if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
                return false;
            // 设置当前线程为独占线程拥有者
            setExclusiveOwnerThread(current);
            // 返回成功
            return true;
        }
```

### tryAcquireShared
```java
        // 尝试获取共享锁
        // 分为三个步骤
        // 1. 如果写锁被其他线程占有，直接返回失败
        // 2. 如果线程不需要阻塞，使用cas更新状态
        // 3. 尝试循环
        protected final int tryAcquireShared(int unused) {
            // 获取当前线程
            Thread current = Thread.currentThread();
            // 获取当前状态
            int c = getState();
            // 当存在写锁并且被其他线程占有时，返回-1
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
            // 共享锁的数量
            int r = sharedCount(c);
            // 当前线程不需要阻塞，并且锁的数量未达到最大值，而且cas更新状态成功
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                // 当锁数量为0时，代表第一次获得锁
                if (r == 0) {
                    // 设置第一次获取读锁的线程为当前线程
                    firstReader = current;
                    // 设置重入数量为1
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    // 重入数量+1
                    firstReaderHoldCount++;
                } else {
                    // 否则获取当前线程的状态进行修改，然后将锁的数量+1
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            // 最后进行循环
            return fullTryAcquireShared(current);
        }
```

### fullTryAcquireShared
```java
        // 自旋获取锁
        final int fullTryAcquireShared(Thread current) {
            HoldCounter rh = null;
            for (;;) {
                // 获取当前状态
                int c = getState();
                // 当有别的线程获得写锁时，返回-1
                if (exclusiveCount(c) != 0) {
                    if (getExclusiveOwnerThread() != current)
                        return -1;
                    // 否则保持独占锁，会阻塞在这里，可能造成死锁？有疑问
                // 当读线程需要阻塞时
                } else if (readerShouldBlock()) {
                    // 第一个获取
                    if (firstReader == current) {
                        // assert firstReaderHoldCount > 0;
                    } else {
                        // 不是当前线程时，需要获取当前线程的状态？有疑问
                        if (rh == null) {
                            rh = cachedHoldCounter;
                            if (rh == null || rh.tid != getThreadId(current)) {
                                rh = readHolds.get();
                                if (rh.count == 0)
                                    readHolds.remove();
                            }
                        }
                        if (rh.count == 0)
                            return -1;
                    }
                }
                // 当锁的数量达到最大值，抛出error
                if (sharedCount(c) == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // 当cas更新状态成功时
                if (compareAndSetState(c, c + SHARED_UNIT)) {
                    // 当前为第一次获取锁，设置第一次获取锁的线程为当前线程，重入次数设置为1
                    if (sharedCount(c) == 0) {
                        firstReader = current;
                        firstReaderHoldCount = 1;
                    } else if (firstReader == current) {
                        // 增加重入次数
                        firstReaderHoldCount++;
                    } else {
                        // 当前线程不是第一次获取读锁的线程时，使用当前线程的计数器
                        if (rh == null)
                            rh = cachedHoldCounter;
                        if (rh == null || rh.tid != getThreadId(current))
                            rh = readHolds.get();
                        else if (rh.count == 0)
                            readHolds.set(rh);
                        rh.count++;
                        cachedHoldCounter = rh; // cache for release
                    }
                    return 1;
                }
            }
        }
```