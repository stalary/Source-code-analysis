### 介绍
- 结构型设计模式
- 对已存在的类进行修饰，扩展功能
- 不改变原结构

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
