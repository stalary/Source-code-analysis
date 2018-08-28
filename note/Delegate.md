### 介绍
- 将A中方法授权给中介，中介直接对外进行交互，外部不知道内部A的存在
-  springmvc框架中的DispatcherServlet是一个典型的委派模式

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