- [介绍](#%E4%BB%8B%E7%BB%8D)
- [桥](#%E6%A1%A5)
- [目标](#%E7%9B%AE%E6%A0%87)
- [来源](#%E6%9D%A5%E6%BA%90)
- [测试类](#%E6%B5%8B%E8%AF%95%E7%B1%BB)
### 介绍
- 结构型设计模式
- 用于连接目标和来源


> 今天通过桥接模式来实现两个地点之间的通行
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
