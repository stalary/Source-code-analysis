### 介绍
- 属于创建型设计模式
- 将一组算法封装起来，可以相互交换

### 策略使用者
```java
public class StrategyManager {

    private ZooStrategy strategy;

    public StrategyManager(ZooStrategy strategy) {
        this.strategy = strategy;
    }

    public void changeStrategy(ZooStrategy strategy) {
        this.strategy = strategy;
    }

    public void execute() {
        strategy.execute();
    }
}
``` 

### 抽象策略
```java
public interface ZooStrategy {

    void execute();
}
```

### 具体策略
```java
public class ElephantStrategy implements ZooStrategy {
    @Override
    public void execute() {
        System.out.println("I am elephant");
    }
}
```

```java
public class BirdStrategy implements ZooStrategy {
    @Override
    public void execute() {
        System.out.println("I am bird");
    }
}
```

```java
public class MonkeyStrategy implements ZooStrategy {
    @Override
    public void execute() {
        System.out.println("I am monkey");
    }
}
```

```java
// 测试类
public class Main {

    public static void main(String[] args) {
        System.out.println("go to zoo to see animal");
        System.out.println("to see elephant");
        StrategyManager manager = new StrategyManager(new ElephantStrategy());
        manager.execute();
        System.out.println("to see bird");
        manager.changeStrategy(new BirdStrategy());
        manager.execute();
        System.out.println("to see monkey");
        manager.changeStrategy(new MonkeyStrategy());
        manager.execute();
    }
}
```

```java
// 测试结果
go to zoo to see animal
to see elephant
I am elephant
to see bird
I am bird
to see monkey
I am monkey
```

