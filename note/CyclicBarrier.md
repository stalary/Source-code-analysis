- [介绍](#%E4%BB%8B%E7%BB%8D)
- [常量&变量](#%E5%B8%B8%E9%87%8F%E5%8F%98%E9%87%8F)
- [await](#await)
- [isBroken](#isbroken)
- [reset](#reset)
- [getNumberWaiting](#getnumberwaiting)
### 介绍
- 用于设置屏障，当一组线程到达时被阻塞，待所有线程到达时，取消阻塞
- 用于多线程计算，最后合并结果

### 常量&变量
```java
    // 共享锁
    private final ReentrantLock lock = new ReentrantLock();

    // 通过锁来获取状态变量
    private final Condition trip = lock.newCondition();

    // 总的等待线程数量(通过构造器传入)
    private final int parties;

    // 当屏障正常打开后运行的程序，通过最后一个调用await的线程来执行
    private final Runnable barrierCommand;

    // 当前的Generation。每当屏障失效或者开闸之后都会自动替换掉。从而实现重置的功能。
    private Generation generation = new Generation();

    // 用来实现重置功能
    private static class Generation {
        boolean broken = false;
    }

    // 还在等待的线程数量
    private int count;
```

### await
```java
    // 线程进入等待
    public int await() throws InterruptedException, BrokenBarrierException {
        try {
            //  不设置超时进入等待，不会抛出超时等待的异常
            return dowait(false, 0L);
        } catch (TimeoutException toe) {
            throw new Error(toe); // cannot happen
        }
    }
```

```java
    private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        // 获取共享锁
        final ReentrantLock lock = this.lock;
        // 加锁(加锁一般都在try之外，否则获取锁出现异常，锁会无故释放)
        lock.lock();
        try {
            // 获取屏障
            final Generation g = generation;

            // 屏障被打破后抛出异常
            if (g.broken)
                throw new BrokenBarrierException();
            // 线程被中断后，打破屏障，抛出中断异常
            if (Thread.interrupted()) {
                breakBarrier();
                throw new InterruptedException();
            }
            // 还在等待的数量减1
            int index = --count;
            // 为0时代表已经到达屏障
            if (index == 0) { 
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    if (command != null)
                        // 执行线程
                        command.run();
                    ranAction = true;
                    // 更新屏障状态并且唤醒所有线程
                    nextGeneration();
                    return 0;
                } finally {
                    if (!ranAction)
                        // 未运行时，直接打破屏障
                        breakBarrier();
                }
            }

            // 循环直到屏障到达，或者中断，销毁屏障，超时
            for (;;) {
                try {
                    // 未超时时继续等待
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        // 设置超时时间时，进行更新
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    // 出现异常
                    // 当屏障存在且未被打破时，打破屏障
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // 否则中断线程
                        Thread.currentThread().interrupt();
                    }
                }
                // 被打破屏障时，抛出异常
                if (g.broken)
                    throw new BrokenBarrierException();
                // 屏障未打破时，返回当前的数量
                if (g != generation)
                    return index;
                // 当超时时，打破屏障，抛出超时异常
                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            // 最后解锁
            lock.unlock();
        }
    }
```

### isBroken
```java
    // 判断屏障是否已经打破
    public boolean isBroken() {
        final ReentrantLock lock = this.lock;
        // 加锁
        lock.lock();
        try {
            // 返回屏障状态
            return generation.broken;
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```

### reset
```java
    // 重置屏障
    public void reset() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // 打破当前屏障
            breakBarrier();
            // 更新屏障
            nextGeneration();
        } finally {
            lock.unlock();
        }
    }
```

### getNumberWaiting
```java
    // 获取正在等待的线程数量
    public int getNumberWaiting() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // 总数量减去当前剩余数量
            return parties - count;
        } finally {
            lock.unlock();
        }
    }
```