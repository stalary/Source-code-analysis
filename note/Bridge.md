### 介绍
- 结构型设计模式
- 用于连接目标和来源

### 桥
```java
public interface Bridge {

    void goTarget();
}
```

### 目标
```java
public class Target implements Bridge {
    @Override
    public void goTarget() {
        System.out.println("我要去目的地");
    }
}
```

### 来源
```java
// 接口
public interface Area {

    void fromArea();
}
```

```java
// 实现类，引入桥，执行目标方法
public class Origin implements Area {

    private Bridge bridge;

    public Origin(Bridge bridge) {
        this.bridge = bridge;
    }

    @Override
    public void fromArea() {
        System.out.println("从原地点出发");
        bridge.goTarget();
    }
}
```

### 测试类
```java
public class Main {

    public static void main(String[] args) {
        Area area = new Origin(new Target());
        area.fromArea();
    }
}
```

```java
// 测试结果
从原地点出发
我要去目的地
```
