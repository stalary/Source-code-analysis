- [介绍](#%E4%BB%8B%E7%BB%8D)
- [回调方法](#%E5%9B%9E%E8%B0%83%E6%96%B9%E6%B3%95)
- [任务抽象类](#%E4%BB%BB%E5%8A%A1%E6%8A%BD%E8%B1%A1%E7%B1%BB)
- [实现类](#%E5%AE%9E%E7%8E%B0%E7%B1%BB)
### 介绍
- 用于java函数式编程
- 解决了同步阻塞的问题

### 回调方法
```java
public interface CallBack {
    void call();
}
```

### 任务抽象类
```java
public abstract class Task {
    // 执行实现的方法，并调用回调
    public final void execute(CallBack callBack) {
        execute();
        callBack.call();
    }

    public abstract void execute();
}
```

### 实现类
```java
public class ZooTask extends Task {
    @Override
    public void execute() {
        System.out.println("execute zoo task start");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
// 测试方法
public class Main {
    public static void main(String[] args) {
        Task task = new ZooTask();
        StopWatch sw = new StopWatch();
        sw.start("task");
        task.execute(() -> System.out.println("execute finish"));
        sw.stop();
        System.out.println(sw.prettyPrint());
    }
}
```

```java
// 测试结果
execute zoo task start
execute finish
StopWatch '': running time (millis) = 1062
-----------------------------------------
ms     %     Task name
-----------------------------------------
01062  100%  task
```