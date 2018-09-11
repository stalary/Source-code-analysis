
### 介绍
- 实现算法的不可变部分，将可变行为留给子类实现
- 可避免代码的重复
- 属于行为型设计模式
- 模版方法在Servlet中运用在Servlet中service方法中的多个Do方法中

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