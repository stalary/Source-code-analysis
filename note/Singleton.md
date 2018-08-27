
### 介绍
- 属于创建型模式
- 保证一个类只有一个实例，并提供一个访问方法
- 仅描述了线程安全的几种方式

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


