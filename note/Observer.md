- [介绍](#%E4%BB%8B%E7%BB%8D)
- [观察者接口](#%E8%A7%82%E5%AF%9F%E8%80%85%E6%8E%A5%E5%8F%A3)
- [具体观察者](#%E5%85%B7%E4%BD%93%E8%A7%82%E5%AF%9F%E8%80%85)
- [被观察者接口](#%E8%A2%AB%E8%A7%82%E5%AF%9F%E8%80%85%E6%8E%A5%E5%8F%A3)
- [具体被观察者](#%E5%85%B7%E4%BD%93%E8%A2%AB%E8%A7%82%E5%AF%9F%E8%80%85)
- [客户端](#%E5%AE%A2%E6%88%B7%E7%AB%AF)
### 介绍
- 属于行为型设计模式
- 优点：观察者与被观察者松耦合，可以进行广播
- 缺点：通知所有观察者消耗较多时间，可能引发循环依赖
- 观察者模式应用在 spring 的事件驱动模型中
  
### 观察者接口
```java
public interface Observer {

    void update(String message);
}
```

### 具体观察者
```java
public class User implements Observer {

    private String name;

    private String message;

    public User(String name) {
        this.name = name;
    }

    @Override
    public void update(String message) {
        this.message = message;
        show();
    }

    public void show() {
        System.out.println(this.name + " receive message {" + this.message + "}");
    }
}
```

### 被观察者接口
```java
public interface Observed {

    void registerObserver(Observer observer);

    void removeObserver(Observer observer);

    void notifyObserver();
}
```

### 具体被观察者
```java
public class MessageServer implements Observed {

    private List<Observer> list = new ArrayList<>();

    private String message;

    @Override
    public void registerObserver(Observer observer) {
        list.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        list.remove(observer);
    }

    @Override
    public void notifyObserver() {
        list.forEach(observer -> observer.update(this.message));
    }

    public void push(String message) {
        this.message = message;
        notifyObserver();
    }
}
```

### 客户端
```java
public class Main {

    public static void main(String[] args) {
        MessageServer server = new MessageServer();
        server.registerObserver(new User("stalary"));
        server.registerObserver(new User("hawk"));
        User claire = new User("claire");
        server.registerObserver(claire);
        server.push("The course will start");
        server.removeObserver(claire);
        server.push("The course is over");
    }
}
```

```java
// 测试结果
stalary receive message {The course will start}
hawk receive message {The course will start}
claire receive message {The course will start}
stalary receive message {The course is over}
hawk receive message {The course is over}
```