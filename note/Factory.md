- [介绍](#%E4%BB%8B%E7%BB%8D)
- [简单工厂](#%E7%AE%80%E5%8D%95%E5%B7%A5%E5%8E%82)
- [工厂方法](#%E5%B7%A5%E5%8E%82%E6%96%B9%E6%B3%95)
- [抽象工厂](#%E6%8A%BD%E8%B1%A1%E5%B7%A5%E5%8E%82)
### 介绍
- 属于创建型设计模式
- 分为三种
    - 简单工厂
    - 工厂方法
    - 抽象工厂
 
### 简单工厂
- 使用一个抽象类来派生子类
- 扩展性差
- 创造方法通常为静态，所以也叫做静态工厂

```java
// 产品抽象类
public abstract class Animal {

    public abstract void desc();
}
```

```java
// 实现类
public class Dog extends Animal {
    @Override
    public void desc() {
        System.out.println("I am dog");
    }
}
```

```java
// 实现类
public class Elephant extends Animal {
    @Override
    public void desc() {
        System.out.println("I am elephant");
    }
}
```

```java
// 实现类
public class Monkey extends Animal {
    @Override
    public void desc() {
        System.out.println("I am monkey");
    }
}
```

```java
// 工厂类，用于创建对象
public class AnimalFactory {

    public static final int DOG = 1;

    public static final int ELEPHANT = 2;

    public static final int MONKEY = 3;

    // 根据传入的参数创建对象
    public static Animal createAnimal(int type) {
        switch (type) {
            case DOG:
                return new Dog();
            case ELEPHANT:
                return new Elephant();
            case MONKEY:
            default:
                return new Monkey();
        }
    }
}
```

```java
    // 测试方法
    public static void main(String[] args) {
        // 创建第一个对象
        Animal animal1 = AnimalFactory.createAnimal(AnimalFactory.DOG);
        animal1.desc();
        // 创建第二个对象
        Animal animal2 = AnimalFactory.createAnimal(AnimalFactory.ELEPHANT);
        animal2.desc();
    }
```

```java
// 结果
I am dog
I am elephant
```

### 工厂方法
- 增加扩展性
- 工厂父类（接口）负责定义产品对象的公共接口，而子类工厂则负责创建具体的产品对象。
- 将实例化操作延迟到了子类中
  
```java
// 工厂接口
public interface PhoneFactory {
    Phone createPhone();
}
```

```java
// 工厂子类
public class XiaoMiFactory implements PhoneFactory {
    @Override
    public Phone createPhone() {
        return new XiaoMiPhone();
    }
}
```

```java
// 工厂子类
public class AppleFactory implements PhoneFactory {
    @Override
    public Phone createPhone() {
        return new ApplePhone();
    }
}
```

```java
// 工厂子类
public class HuaWeiFactory implements PhoneFactory {
    @Override
    public Phone createPhone() {
        return new HuaWeiPhone();
    }
}
```

```java
// 产品接口
public interface Phone {
    void desc();
}
```

```java
// 产品子类
public class XiaoMiPhone implements Phone {
    @Override
    public void desc() {
        System.out.println("I am xiaomi Phone");
    }
}
```

```java
// 产品子类
public class HuaWeiPhone implements Phone {
    @Override
    public void desc() {
        System.out.println("I am huawei Phone");
    }
}
```

```java
// 产品子类
public class ApplePhone implements Phone {
    @Override
    public void desc() {
        System.out.println("I am Apple Phone");
    }
}
```

```java
// 测试类
public class Main {

    public static void main(String[] args) {
        PhoneFactory factory;
        factory = new AppleFactory();
        Phone apple = factory.createPhone();
        apple.desc();
        factory = new HuaWeiFactory();
        Phone huawei = factory.createPhone();
        huawei.desc();
        factory = new XiaoMiFactory();
        Phone xiaomi = factory.createPhone();
        xiaomi.desc();
    }
}
```

```java
// 测试结果
I am Apple Phone
I am huawei Phone
I am xiaomi Phone
```

### 抽象工厂
- 用于生产产品族
- 扩展困难

```java
// 产品族
public interface Android {
    void desc();
}
```

```java
// 产品族
public interface IOS {
    void desc();
}
```

```java
// 产品实现类
public class HuaWei implements Android {
    @Override
    public void desc() {
        System.out.println("I am huawei Android");
    }
}
```

```java
// 产品实现类
public class XiaoMi implements Android {
    @Override
    public void desc() {
        System.out.println("I am xiaomi Android");
    }
}
```

```java
// 产品实现类
public class Iphone implements IOS {
    @Override
    public void desc() {
        System.out.println("I am iphone ios");
    }
}
```

```java
// 工厂接口
public interface Factory {
    IOS createIOS();
    Android createAndroid();
}
```

```java
// 具体工厂
public class PhoneFactory1 implements Factory{
    @Override
    public IOS createIOS() {
        return new Iphone();
    }

    @Override
    public Android createAndroid() {
        return new HuaWei();
    }
}
```

```java
// 具体工厂
public class PhoneFactory2 implements Factory {
    @Override
    public IOS createIOS() {
        return new Iphone();
    }

    @Override
    public Android createAndroid() {
        return new XiaoMi();
    }
}
```

```java
// 测试方法
public class Main {

    public static void main(String[] args) {
        Factory factory = new PhoneFactory1();
        Android android1 = factory.createAndroid();
        android1.desc();
        IOS ios = factory.createIOS();
        ios.desc();
        factory = new PhoneFactory2();
        Android android2 = factory.createAndroid();
        android2.desc();
    }
}
```

```java
// 测试结果
I am huawei Android
I am iphone ios
I am xiaomi Android
```


