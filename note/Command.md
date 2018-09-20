- [介绍](#%E4%BB%8B%E7%BB%8D)
- [命令抽象类](#%E5%91%BD%E4%BB%A4%E6%8A%BD%E8%B1%A1%E7%B1%BB)
- [具体命令类](#%E5%85%B7%E4%BD%93%E5%91%BD%E4%BB%A4%E7%B1%BB)
- [命令接收者](#%E5%91%BD%E4%BB%A4%E6%8E%A5%E6%94%B6%E8%80%85)
- [命令发送者](#%E5%91%BD%E4%BB%A4%E5%8F%91%E9%80%81%E8%80%85)
- [客户端](#%E5%AE%A2%E6%88%B7%E7%AB%AF)
### 介绍
- 属于行为型设计模式
- 优点：降低耦合度，请求和接收者之间不存在引用，父类抽象算法，子类实现，更加灵活
- 缺点：需要为不同的实现创建一个子类，使系统更加庞大
- 命令模式应用在Tomcat中，HttpConnector发出请求，HttpProcessor作为命令，Container为抽象接收者

### 命令抽象类
```java
public interface Command {

    public void exec();
}
```

### 具体命令类
```java
public class CommandImpl implements Command {
    private Receiver receiver;
    public CommandImpl(Receiver receiver) {
        this.receiver = receiver;
    }
    @Override
    public void exec() {
        receiver.action();
    }
}
```

### 命令接收者
```java
public class Receiver {
    public String name;
    public Receiver(String name) {
        this.name = name;
    }
    public void action() {
        System.out.println(name + " receive command");
    }
}
```

### 命令发送者
```java
public class Invoker {
    private Command command;
    public Invoker(Command command) {
        this.command = command;
    }
    public void action() {
        command.exec();
    }
}
```

### 客户端
```java
public class Main {

    public static void main(String[] args) {
        Receiver receiver = new Receiver("stalary");
        Command command = new CommandImpl(receiver);
        Invoker invoker = new Invoker(command);
        invoker.action();
    }
}
```

```java
// 测试结果
stalary receive command
```