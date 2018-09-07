- [介绍](#%E4%BB%8B%E7%BB%8D)
- [策略使用者](#%E7%AD%96%E7%95%A5%E4%BD%BF%E7%94%A8%E8%80%85)
- [抽象策略](#%E6%8A%BD%E8%B1%A1%E7%AD%96%E7%95%A5)
- [具体策略](#%E5%85%B7%E4%BD%93%E7%AD%96%E7%95%A5)
### 介绍
- 属于创建型设计模式
- 将一组算法封装起来，可以相互交换

> 今天通过策略模式来实现各个动物之间的交换
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

