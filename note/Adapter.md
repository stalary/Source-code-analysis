- [介绍](#%E4%BB%8B%E7%BB%8D)
- [目标接口](#%E7%9B%AE%E6%A0%87%E6%8E%A5%E5%8F%A3)
- [适配器](#%E9%80%82%E9%85%8D%E5%99%A8)
- [需要适配的类](#%E9%9C%80%E8%A6%81%E9%80%82%E9%85%8D%E7%9A%84%E7%B1%BB)
- [客户端](#%E5%AE%A2%E6%88%B7%E7%AB%AF)
### 介绍
- 结构型设计模式
- 对已经存在的老接口做适配以适应新的需求

### 目标接口
```java
// 使用三角插头进行充电
public interface ThreePinPlug {

    void charge();
}
```

### 适配器
```java
public class TwoPinPlugAdapter implements ThreePinPlug {

    private TwoPinPlug plug;

    public TwoPinPlugAdapter() {
        this.plug = new TwoPinPlug();
    }

    @Override
    public void charge() {
        plug.charge();
    }
}
```

### 需要适配的类
```java
public class TwoPinPlug {

    public void charge() {
        System.out.println("Is charging");
    }
}
```

### 客户端
```java
public class User {
    // 使用三脚插头充电，做适配
    private ThreePinPlug plug;

    public User(ThreePinPlug plug) {
        this.plug = plug;
    }

    public void setPlug(ThreePinPlug plug) {
        this.plug = plug;
    }

    public void charge() {
        plug.charge();
    }
}
```

```java
// 测试类
public class Main {

    public static void main(String[] args) {
        User user = new User(new TwoPinPlugAdapter());
        user.charge();
    }
}
```

```java
// 测试结果
Is charging
```