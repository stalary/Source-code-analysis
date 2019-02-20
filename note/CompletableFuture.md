- [介绍](#%E4%BB%8B%E7%BB%8D)
- [supplyAsync](#supplyasync)
- [runAsync](#runasync)
- [thenApply](#thenapply)
- [thenAccept](#thenaccept)
- [thenRun](#thenrun)
- [thenCombine](#thencombine)
### 介绍
CompletableFuture是java8提供的并发api，实现了Future和CompletionStage(异步计算的某一个阶段)，提供了函数式编程的能力，可以通过回调的方式处理计算结果

### supplyAsync
用于创建有返回值的异步任务，可选择是否传入自定义的Executor，不传入则默认使用ForkJoinPool.commonPool()
```java
    // 执行一个有返回值的异步任务
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
        return asyncSupplyStage(asyncPool, supplier);
    }

    // 传入自定义的Executor
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,
                                                       Executor executor) {
        return asyncSupplyStage(screenExecutor(executor), supplier);
    }
```
demo
```java
    public static void main(String[] args) {
        CompletableFuture
                .supplyAsync(() -> cal(1, 3)) // 执行异步任务
                .whenComplete((r, e) -> System.out.println(r)) // 完成时执行
                .join(); // 等待异步任务
    }

    // 等待执行的任务
    private static int cal(int i, int j) {
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return i + j;
    }
```

### runAsync
用于创建无返回值的异步任务，可选择是否传入自定义的Executor，不传入则默认使用ForkJoinPool.commonPool()
```java
    public static CompletableFuture<Void> runAsync(Runnable runnable) {
        return asyncRunStage(asyncPool, runnable);
    }

    public static CompletableFuture<Void> runAsync(Runnable runnable,
                                                   Executor executor) {
        return asyncRunStage(screenExecutor(executor), runnable);
    }
```
demo
```java
    public static void main(String[] args) {
        CompletableFuture
                .runAsync(Main::print) // 执行无返回值的异步任务
                .join(); // 等待任务结束
    }

    // 等待执行的无返回值异步任务
    private static void print() {
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("print");
    }
```

### thenApply
接收上一个任务的返回值用于执行当前任务并返回执行结果
```java
    // 同步执行，使用当前线程
    public <U> CompletableFuture<U> thenApply(
        Function<? super T,? extends U> fn) {
        return uniApplyStage(null, fn);
    }

    // 另起一个线程执行任务
    public <U> CompletableFuture<U> thenApplyAsync(
        Function<? super T,? extends U> fn) {
        return uniApplyStage(asyncPool, fn);
    }

    // 传入自定义的Executor新起一个线程执行任务
    public <U> CompletableFuture<U> thenApplyAsync(
        Function<? super T,? extends U> fn, Executor executor) {
        return uniApplyStage(screenExecutor(executor), fn);
    }
```
demo
```java
    public static void main(String[] args) {
        CompletableFuture
                .supplyAsync(() -> first(2))
                .thenApply(Main::second) // 接收first返回值用来执行second
                .whenComplete((r, e) -> System.out.println(r))
                .join();
    }

    private static int first(int i) {
        return 2 * i;
    }

    private static int second(int i) {
        return i * i;
    }
```

### thenAccept
接收上一个任务的返回值执行任务，无返回值
```java
    public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
        return uniAcceptStage(null, action);
    }

    public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action) {
        return uniAcceptStage(asyncPool, action);
    }

    public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action,
                                                   Executor executor) {
        return uniAcceptStage(screenExecutor(executor), action);
    }
```
demo
```java
    public static void main(String[] args) {
        CompletableFuture.supplyAsync(() -> first(3))
                .thenAccept(System.out::println) // 打印出上一个任务的返回值
                .join();
    }

    private static int first(int i) {
        return 2 * i;
    }
```

### thenRun
完成上一个任务时执行，不接收上一个任务返回值，同时也不会返回结果
```java
    public CompletableFuture<Void> thenRun(Runnable action) {
        return uniRunStage(null, action);
    }

    public CompletableFuture<Void> thenRunAsync(Runnable action) {
        return uniRunStage(asyncPool, action);
    }

    public CompletableFuture<Void> thenRunAsync(Runnable action,
                                                Executor executor) {
        return uniRunStage(screenExecutor(executor), action);
    }
```
demo
```java
    public static void main(String[] args) {
        CompletableFuture.supplyAsync(() -> first(3))
                .thenRun(() -> test(2))
                .join();
    }

    private static void test(int i) {
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(i + " in async method");
    }
```

### thenCombine
整合两个CompletableFuture的结果
```java
    public <U,V> CompletableFuture<V> thenCombine(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn) {
        return biApplyStage(null, other, fn);
    }

    public <U,V> CompletableFuture<V> thenCombineAsync(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn) {
        return biApplyStage(asyncPool, other, fn);
    }

    public <U,V> CompletableFuture<V> thenCombineAsync(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn, Executor executor) {
        return biApplyStage(screenExecutor(executor), other, fn);
    }
```
demo
```java
    public static void main(String[] args) {
        CompletableFuture.supplyAsync(() -> first(3))
                .thenApply(i -> "cur Integer is " + i) // 执行带有返回值的任务
                .thenCombine(CompletableFuture.supplyAsync(() -> second(2)), (s1, s2) -> s1 + ":" + s2) // 接收上一个任务的返回值，并与当前任务返回值合并
                .thenAccept(System.out::println) // 接收上一个任务的返回值并打印
                .join();
    }

    private static int first(int i) {
        return 2 * i;
    }

    private static int second(int i) {
        return i * i;
    }
```