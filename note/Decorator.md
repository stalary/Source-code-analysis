- [介绍](#%E4%BB%8B%E7%BB%8D)
- [组件类](#%E7%BB%84%E4%BB%B6%E7%B1%BB)
- [实现类](#%E5%AE%9E%E7%8E%B0%E7%B1%BB)
- [修饰类](#%E4%BF%AE%E9%A5%B0%E7%B1%BB)
- [测试类和结果](#%E6%B5%8B%E8%AF%95%E7%B1%BB%E5%92%8C%E7%BB%93%E6%9E%9C)
### 介绍
- 属于结构型设计模式
- 优点：动态给对象添加功能，比生成子类更加灵活
- 缺点：增加类的数量
- 装饰器模式应用在 IO 中的 Buffered
- 装饰器模式应用在 filter 中

> 今天通过装饰器模式来实现一个披着羊皮的狼
### 组件类
```java
public interface Animal {
    void shout();
    void appearance();
}
```

### 实现类
```java
public class Wolf implements Animal {
    @Override
    public void shout() {
        System.out.println("I am a Wolf");
    }

    @Override
    public void appearance() {
        System.out.println("I have gray fur");
    }
}
```

### 修饰类
```java
public class SheepWolf implements Animal {

    Animal animal;

    public SheepWolf(Animal animal) {
        this.animal = animal;
    }

    @Override
    public void shout() {
        animal.shout();
    }

    @Override
    public void appearance() {
        animal.appearance();
        System.out.println("I also have sheepskin");
    }
}
```

### 测试类和结果
```java
// 测试类
public class Main {

    public static void main(String[] args) {
        Animal animal = new SheepWolf(new Wolf());
        animal.shout();
        animal.appearance();
    }
}
```

```java
// 结果
I am a Wolf
I have gray fur
I also have sheepskin
```
