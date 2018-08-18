### 介绍
- 三种模式
    - 写
    - 读
    - 乐观读
- 基于CLH自旋锁实现，保证公平性，提供乐观读策略，不阻塞写线程
- 由版本和模式组成
- 读写锁的优化(读少写多的情况下，读写锁会发生写入线程的饥饿问题)

### 常量
```java
// 当前可用的cpu数量，用于控制自旋
private static final int NCPU = Runtime.getRuntime().availableProcessors();

// 入队自旋次数
private static final int SPINS = (NCPU > 1) ? 1 << 6 : 0;

// 阻塞在头节点的自旋次数
private static final int HEAD_SPINS = (NCPU > 1) ? 1 << 10 : 0;

// 重新进入阻塞的自旋次数
private static final int MAX_HEAD_SPINS = (NCPU > 1) ? 1 << 16 : 0;

// 自旋锁让出的时机，必须是2的n次幂-1
private static final int OVERFLOW_YIELD_RATE = 7;

// 溢出前读锁的数量
private static final int LG_READERS = 7;

// 锁的状态和戳的操作
private static final long RUNIT = 1L;
private static final long WBIT  = 1L << LG_READERS;
private static final long RBITS = WBIT - 1L;
private static final long RFULL = RBITS - 1L;
private static final long ABITS = RBITS | WBIT;
private static final long SBITS = ~RBITS; // note overlap with ABITS

// 锁的初始状态，避免0的失败状态
private static final long ORIGIN = WBIT << 1;

// 取消获取锁方法的特殊值
private static final long INTERRUPTED = 1L;

// 节点的状态
private static final int WAITING   = -1;
private static final int CANCELLED =  1;

// 节点的模式，使用int允许计算
private static final int RMODE = 0;
private static final int WMODE = 1;

// CLH队列的头
private transient volatile WNode whead;

// CLH队列的尾
private transient volatile WNode wtail;

// 锁的状态
private transient volatile long state;

// 额外的读锁数量
private transient int readerOverflow;
```

### writeLock
```java
    // 获取排他的写锁，获取失败会一直阻塞直到成功，返回的戳可以用于解锁和转化模式
    public long writeLock() {
        long s, next;  // bypass acquireWrite in fully unlocked case only
        // 使用cas尝试修改锁的状态，否则调用acquireWrite
        return ((((s = state) & ABITS) == 0L &&
                 U.compareAndSwapLong(this, STATE, s, next = s + WBIT)) ?
                next : acquireWrite(false, 0L));
    }
```

### tryWriteLock
```java
    // 尝试获取写锁，失败直接返回0
    public long tryWriteLock() {
        long s, next;
        return ((((s = state) & ABITS) == 0L &&
                 U.compareAndSwapLong(this, STATE, s, next = s + WBIT)) ?
                next : 0L);
    }
```

### readLock
```java
    // 非排他的获取读锁，当获取失败时阻塞直到成功
    public long readLock() {
        long s = state, next;  // bypass acquireRead on common uncontended case
        return ((whead == wtail && (s & ABITS) < RFULL &&
                 U.compareAndSwapLong(this, STATE, s, next = s + RUNIT)) ?
                next : acquireRead(false, 0L));
    }
```

### tryReadLock
```java
    // 尝试获取读锁
    public long tryReadLock() {
        // 使用自旋+cas获取锁，失败则返回0
        for (;;) {
            long s, m, next;
            if ((m = (s = state) & ABITS) == WBIT)
                return 0L;
            else if (m < RFULL) {
                if (U.compareAndSwapLong(this, STATE, s, next = s + RUNIT))
                    return next;
            }
            else if ((next = tryIncReaderOverflow(s)) != 0L)
                return next;
        }
    }
```

### tryOptimisticRead
```java
    // 获取乐观读锁，直接返回戳，后面可以进行验证
    public long tryOptimisticRead() {
        long s;
        // 有写锁返回0，否则返回256
        return (((s = state) & WBIT) == 0L) ? (s & SBITS) : 0L;
    }
```

### validate
```java
    // 验证从获取乐观锁开始到现在有无写锁，如果有写锁则返回false，否则返回true
    public boolean validate(long stamp) {
        U.loadFence();
        return (stamp & SBITS) == (state & SBITS);
    }
```

### unlock
```java
    // 释放读锁或者写锁
    public void unlock(long stamp) {
        long a = stamp & ABITS, m, s; WNode h;
        // 匹配戳
        while (((s = state) & SBITS) == (stamp & SBITS)) {
            // 当为初始状态时直接跳出
            if ((m = s & ABITS) == 0L)
                break;
            // 为写锁时，进行释放锁操作
            else if (m == WBIT) {
                if (a != m)
                    break;
                state = (s += WBIT) == 0L ? ORIGIN : s;
                if ((h = whead) != null && h.status != 0)
                    release(h);
                return;
            }
            // 没有锁
            else if (a == 0L || a >= WBIT)
                break;
            // 读锁时，进行cas释放锁
            else if (m < RFULL) {
                if (U.compareAndSwapLong(this, STATE, s, s - RUNIT)) {
                    if (m == RUNIT && (h = whead) != null && h.status != 0)
                        release(h);
                    return;
                }
            }
            // 减少readerOverflow
            else if (tryDecReaderOverflow(s) != 0L)
                return;
        }
        // 不匹配，直接抛出异常
        throw new IllegalMonitorStateException();
    }
```

### unlockWrite
```java
    // 释放写锁
    public void unlockWrite(long stamp) {
        WNode h;
        // 当未持有写锁时，抛出异常
        if (state != stamp || (stamp & WBIT) == 0L)
            throw new IllegalMonitorStateException();
        state = (stamp += WBIT) == 0L ? ORIGIN : stamp;
        // 判断成功后释放锁
        if ((h = whead) != null && h.status != 0)
            release(h);
    }
```

### tryUnlockWrite 
```java
    // 尝试释放写锁，如果持有写锁，释放，返回true，否则返回false
    public boolean tryUnlockWrite() {
        long s; WNode h;
        if (((s = state) & WBIT) != 0L) {
            state = (s += WBIT) == 0L ? ORIGIN : s;
            if ((h = whead) != null && h.status != 0)
                release(h);
            return true;
        }
        return false;
    }
```

### unlockRead
```java
    // 释放读锁
    public void unlockRead(long stamp) {
        long s, m; WNode h;
        // 通过死循环+cas来释放锁
        for (;;) {
            // 状态不匹配时，直接抛出异常
            if (((s = state) & SBITS) != (stamp & SBITS) ||
                (stamp & ABITS) == 0L || (m = s & ABITS) == 0L || m == WBIT)
                throw new IllegalMonitorStateException();
            if (m < RFULL) {
                // cas修改状态
                if (U.compareAndSwapLong(this, STATE, s, s - RUNIT)) {
                    // 修改成功后释放锁
                    if (m == RUNIT && (h = whead) != null && h.status != 0)
                        release(h);
                    break;
                }
            }
            // 减少readerOverflow
            else if (tryDecReaderOverflow(s) != 0L)
                break;
        }
    }
```

### tryUnlockRead
```java
    // 尝试释放读锁，如果持有读锁，释放读锁，返回true，否则返回false
    public boolean tryUnlockRead() {
        long s, m; WNode h;
        while ((m = (s = state) & ABITS) != 0L && m < WBIT) {
            // 当读锁未满时，cas进行释放锁
            if (m < RFULL) {
                if (U.compareAndSwapLong(this, STATE, s, s - RUNIT)) {
                    if (m == RUNIT && (h = whead) != null && h.status != 0)
                        release(h);
                    return true;
                }
            }
            // 否则减少readerOverflow
            else if (tryDecReaderOverflow(s) != 0L)
                return true;
        }
        return false;
    }
```

### tryConvertToWriteLock
```java
    // 尝试转化为写锁
    public long tryConvertToWriteLock(long stamp) {
        long a = stamp & ABITS, m, s, next;
        while (((s = state) & SBITS) == (stamp & SBITS)) {
            // 没有锁
            if ((m = s & ABITS) == 0L) {
                if (a != 0L)
                    break;
                // 使用cas来持有锁
                if (U.compareAndSwapLong(this, STATE, s, next = s + WBIT))
                    return next;
            }
            // 持有写锁
            else if (m == WBIT) {
                if (a != m)
                    // 表示其他线程持有，直接返回0，代表转化失败
                    break;
                // 返回当前戳
                return stamp;
            }
            // 持有读锁
            else if (m == RUNIT && a != 0L) {
                // cas释放读锁，并转化为写锁
                if (U.compareAndSwapLong(this, STATE, s,
                                         next = s - RUNIT + WBIT))
                    return next;
            }
            else
                break;
        }
        return 0L;
    }
```

### tryConvertToReadLock
```java
    // 尝试转化为读锁
    public long tryConvertToReadLock(long stamp) {
        long a = stamp & ABITS, m, s, next; WNode h;
        while (((s = state) & SBITS) == (stamp & SBITS)) {
            // 没有锁
            if ((m = s & ABITS) == 0L) {
                if (a != 0L)
                    // 其他线程有锁时，直接返回0
                    break;
                // 当读锁数量未满时，cas持有读锁
                else if (m < RFULL) {
                    if (U.compareAndSwapLong(this, STATE, s, next = s + RUNIT))
                        return next;
                }
                // 否则增加readerOverflow，持有读锁
                else if ((next = tryIncReaderOverflow(s)) != 0L)
                    return next;
            }
            // 持有写锁
            else if (m == WBIT) {
                if (a != m)
                    // 非当前线程持有，返回0
                    break;
                // 释放写锁，持有读锁
                state = next = s + (WBIT + RUNIT);
                if ((h = whead) != null && h.status != 0)
                    release(h);
                return next;
            }
            // 持有读锁，直接返回(读锁为共享锁，所以不用判断持有者)
            else if (a != 0L && a < WBIT)
                return stamp;
            else
                break;
        }
        return 0L;
    }
```

### tryConvertToOptimisticRead
```java
    // 尝试转化为乐观读锁，也可以作为尝试解锁
    public long tryConvertToOptimisticRead(long stamp) {
        long a = stamp & ABITS, m, s, next; WNode h;
        U.loadFence();
        for (;;) {
            if (((s = state) & SBITS) != (stamp & SBITS))
                break;
            // 当没有锁时
            if ((m = s & ABITS) == 0L) {
                if (a != 0L)
                    // 其他线程持有锁时直接返回0
                    break;
                // 返回戳
                return s;
            }
            // 当持有写锁时
            else if (m == WBIT) {
                if (a != m)
                    // 其他线程持有时直接返回0
                    break;
                // 释放写锁，返回戳
                state = next = (s += WBIT) == 0L ? ORIGIN : s;
                if ((h = whead) != null && h.status != 0)
                    release(h);
                return next;
            }
            // 其他线程持有锁时，直接返回0
            else if (a == 0L || a >= WBIT)
                break;
            // 当读锁数量不满时
            else if (m < RFULL) {
                // 使用cas释放读锁，返回戳(悲观读锁转化为乐观读锁)
                if (U.compareAndSwapLong(this, STATE, s, next = s - RUNIT)) {
                    if (m == RUNIT && (h = whead) != null && h.status != 0)
                        release(h);
                    return next & SBITS;
                }
            }
            else if ((next = tryDecReaderOverflow(s)) != 0L)
                return next & SBITS;
        }
        return 0L;
    }
```

```java
    // 当读锁满时，将readerOverflow增加，否则直接返回戳
    private long tryIncReaderOverflow(long s) {
        // 读锁数量到达临界点
        if ((s & ABITS) == RFULL) {
            // cas获取读锁
            if (U.compareAndSwapLong(this, STATE, s, s | RBITS)) {
                // 增加readerOverflow
                ++readerOverflow;
                state = s;
                return s;
            }
        }
        // 生成随机数
        else if ((LockSupport.nextSecondarySeed() &
                  OVERFLOW_YIELD_RATE) == 0)
            // 放弃cpu资源
            Thread.yield();
        // 当未满时，返回0，代表增加溢出变量失败
        return 0L;
    }
```

