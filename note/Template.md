
### 介绍
- 属于行为型设计模式
- 优点：去除子类重复代码，子类可实现算法细节进行扩展，符合开放-封闭原则
- 缺点：每个不同的实现都需要定义一个子类，导致类的数量增加
- 模版方法在运用在 Servlet 中的多个 do 方法中
- 模版方法也应用于 Spring ioc 容器的初始化

> 今天通过模版方法来实现短信的发送
### 抽象模版
```java
public abstract class Note {
    public abstract String content();

    public void send(String receiver) {
        System.out.println("尊敬的" + receiver);
        System.out.println(content());
    }
}
```

### 具体模版
```java
// 验证码
public class Code extends Note {

    @Override
    public String content() {
        return "您的验证码为52084";
    }
}
```

```java
// 警告
public class Warn extends Note {
    @Override
    public String content() {
        return "您的账号在济南登陆";
    }
}
```

```java
// 测试方法
public class Main {

    public static void main(String[] args) {
        Note code = new Code();
        code.send("stalary");
        System.out.println("----------------");
        Note warn = new Warn();
        warn.send("stalary");
    }
}
```

```java
// 测试结果
尊敬的stalary
您的验证码为52084
----------------
尊敬的stalary
您的账号在济南登陆
```