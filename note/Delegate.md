- [介绍](#%E4%BB%8B%E7%BB%8D)
- [动作](#%E5%8A%A8%E4%BD%9C)
- [控制器](#%E6%8E%A7%E5%88%B6%E5%99%A8)
- [委派者](#%E5%A7%94%E6%B4%BE%E8%80%85)
### 介绍
- 不属于23种设计模式
- 将请求交给其他对象来处理
- springmvc框架中的DispatcherServlet是一个典型的委派模式
-  与代理的区别，委派是一个对象引用了另一个对象，而代理是若干对象实现了共同接口

### 动作
```java
// 首先指定一个提供模版的接口，即为要实现的目标
public interface Zoo {

    void print(String message);
}
```

```java
// 实现接口，即实现的方式
public class Elephant implements Zoo {
    @Override
    public void print(String message) {
        System.out.println("I am a elephant: " + message);
    }
}
```

```java
public class Monkey implements Zoo {
    @Override
    public void print(String message) {
        System.out.println("I am a monkey: " + message);
    }
}
```

### 控制器
```java
public class ZooController implements Zoo {

    private final Zoo zoo;

    public ZooController(Zoo zoo) {
        this.zoo = zoo;
    }


    @Override
    public void print(String message) {
        zoo.print(message);
    }
}
```

### 委派者
```java
// 选择对应的方式实现目标
public class Main {

    public static void main(String[] args) {
        ZooController elephant = new ZooController(new Elephant());
        ZooController monkey = new ZooController(new Monkey());
        elephant.print("hello world");
        monkey.print("hello world");
    }
}
```

```java
// 结果
I am a elephant: hello world
I am a monkey: hello world
```