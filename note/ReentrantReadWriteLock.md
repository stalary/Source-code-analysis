- [介绍](#%E4%BB%8B%E7%BB%8D)
- [常量和变量](#%E5%B8%B8%E9%87%8F%E5%92%8C%E5%8F%98%E9%87%8F)
- [同步器](#%E5%90%8C%E6%AD%A5%E5%99%A8)
- [tryRelease](#tryrelease)
- [tryReleaseShared](#tryreleaseshared)
- [tryAcquire](#tryacquire)
- [tryAcquireShared](#tryacquireshared)
- [fullTryAcquireShared](#fulltryacquireshared)
- [tryWriteLock](#trywritelock)
- [tryReadLock](#tryreadlock)
- [NonfairSync](#nonfairsync)
- [FairSync](#fairsync)
- [ReadLock](#readlock)
- [WriteLock](#writelock)
- [isFair](#isfair)
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
                // firstReader不是当前线程时，更新缓存
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
                    // firstReader不是当前线程时，记录每一个线程读锁的数量
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
                // 当前线程不是写锁的持有者，返回-1
                if (exclusiveCount(c) != 0) {
                    if (getExclusiveOwnerThread() != current)
                        return -1;
                // 出现持有写锁，申请写锁的操作，而且其他线程也在申请写锁，只能结束然后去排队，否则造成死锁
                } else if (readerShouldBlock()) {
                    // 第一个获取
                    if (firstReader == current) {
                        // assert firstReaderHoldCount > 0;
                    } else {
                        // firstReader不是当前线程时，更新数量
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
                        // firstReader不是当前线程时，更新数量
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

### tryWriteLock
```java
        // 尝试获取写锁
        final boolean tryWriteLock() {
            // 获取当前线程
            Thread current = Thread.currentThread();
            // 获取当前状态
            int c = getState();
            // 存在锁
            if (c != 0) {
                // 不存在写锁(存在读锁，读写互斥)或者当前线程不是独占线程时获取失败
                int w = exclusiveCount(c);
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                // 写锁达到最大数量时，抛出error
                if (w == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
            }
            // 当不存在锁时，cas成功更新状态时获取成功
            if (!compareAndSetState(c, c + 1))
                return false;
            setExclusiveOwnerThread(current);
            return true;
        }
```

### tryReadLock
```java
        // 尝试获取读锁
         // 与tryAcquireShared基本一致，只是没有判断readerShouldBlock，直接采用非公平模式
        final boolean tryReadLock() {
            Thread current = Thread.currentThread();
            for (;;) {
                int c = getState();
                if (exclusiveCount(c) != 0 &&
                    getExclusiveOwnerThread() != current)
                    return false;
                int r = sharedCount(c);
                if (r == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                if (compareAndSetState(c, c + SHARED_UNIT)) {
                    if (r == 0) {
                        firstReader = current;
                        firstReaderHoldCount = 1;
                    } else if (firstReader == current) {
                        firstReaderHoldCount++;
                    } else {
                        HoldCounter rh = cachedHoldCounter;
                        if (rh == null || rh.tid != getThreadId(current))
                            cachedHoldCounter = rh = readHolds.get();
                        else if (rh.count == 0)
                            readHolds.set(rh);
                        rh.count++;
                    }
                    return true;
                }
            }
        }
```

### NonfairSync
```java
    // 非公平同步器，这个非公平策略的同步器是写锁优先的，申请写锁时总是不阻塞
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = -8159625535654395037L;
        final boolean writerShouldBlock() {
            // 写锁可以直接进入
            return false;
        }
        final boolean readerShouldBlock() {
            // 用于避免写线程饥饿，如果线程临时出现在等待队列的头部则阻塞
            return apparentlyFirstQueuedIsExclusive();
        }
    }
```

### FairSync
```java
    // 公平同步器，如果线程准备获取锁时，同步队列里有等待线程，则阻塞获取锁，不管是否是重入
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -2274990926593161451L;
        final boolean writerShouldBlock() {
            // 获取前继节点
            return hasQueuedPredecessors();
        }
        final boolean readerShouldBlock() {
            // 获取前继节点
            return hasQueuedPredecessors();
        }
    }
```

### ReadLock
```java
    // 读锁
    public static class ReadLock implements Lock, java.io.Serializable {
        // 获取读锁
        public void lock() {
            sync.acquireShared(1);
        }
        // 尝试获取读锁(非公平)
        public boolean tryLock() {
            return sync.tryReadLock();
        }
        // 释放读锁
        public void unlock() {
            sync.releaseShared(1);
        }
    }
```

### WriteLock
```java
    // 写锁
    public static class WriteLock implements Lock, java.io.Serializable {
        // 获取写锁
        public void lock() {
            sync.acquire(1);
        }
        // 尝试获取写锁(非公平)
        public boolean tryLock( ) {
            return sync.tryWriteLock();
        }
        // 释放写锁
        public void unlock() {
            sync.release(1);
        }
    }
```

### isFair
```java
    // 判断当前是否是公平锁
    public final boolean isFair() {
        return sync instanceof FairSync;
    }
```

