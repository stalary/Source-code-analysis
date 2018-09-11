- [介绍](#%E4%BB%8B%E7%BB%8D)
- [枚举](#%E6%9E%9A%E4%B8%BE)
- [双重检测](#%E5%8F%8C%E9%87%8D%E6%A3%80%E6%B5%8B)
- [内部类](#%E5%86%85%E9%83%A8%E7%B1%BB)
### 介绍
- 属于创建型设计模式
- 优点：确保所有的对象都访问一个实例，避免对共享资源的多重占用
- 缺点：不适用于变化的对象，扩展困难
- 单例模式应用于Spring中Bean的创建

### 枚举
构造单例最简单的方法，就是使用枚举

使用枚举可以保证线程安全
```java
// 枚举
public enum ZooEnum {
    Instance;

    @Override
    public String toString() {
        return getDeclaringClass().getCanonicalName() + "@" + hashCode();
    }
}
```

### 双重检测
// 线程安全，懒加载
```java
public class ZooDoubleCheck {

    private static volatile ZooDoubleCheck instance;

    private ZooDoubleCheck() {
        // 防止通过反射调用实例化
        if (instance != null) {
            throw new IllegalStateException("Already initialized.");
        }
    }

    public static ZooDoubleCheck getInstance() {
        // 使用局部变量会加快速度
        ZooDoubleCheck check = instance;
        if (check == null) {
            synchronized (ZooDoubleCheck.class) {
                // 更新局部变量
                check = instance;
                if (check == null) {
                    // 双重检测都为null时进行实例化
                    instance = check = new ZooDoubleCheck();
                }
            }
        }
        return check;
    }
}
```

### 内部类
线程安全，懒加载
```java
public class ZooInner {

    // 私有构造方法
    private ZooInner() {
    }

    public static ZooInner getInstance() {
        return Holder.INSTANCE;
    }

    private static class Holder {
        private static final ZooInner INSTANCE = new ZooInner();
    }
}
```


