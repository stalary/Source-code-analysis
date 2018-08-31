- [介绍](#%E4%BB%8B%E7%BB%8D)
- [算法策略](#%E7%AE%97%E6%B3%95%E7%AD%96%E7%95%A5)
- [模版方法](#%E6%A8%A1%E7%89%88%E6%96%B9%E6%B3%95)
### 介绍
- 实现算法的不可变部分，将可变行为留给子类实现
- 可避免代码的重复
- 属于行为型设计模式

### 算法策略
```java
public abstract class Animal {

    protected abstract String introduce();

    protected abstract void shout();

    protected abstract void doWork();

    // 使用的算法
    public void see() {
        System.out.println(introduce() + " discover you");
        shout();
        doWork();
    }
}
```

### 模版方法
```java
// 抽象模版
public class Zoo {

    // 引入算法
    private Animal animal;

    public Zoo(Animal animal) {
        this.animal = animal;
    }

    public void see() {
        animal.see();
    }

    // 提供改变可变算法的方法
    public void changeAnimal(Animal animal) {
        this.animal = animal;
    }
}
```

```java
// 实现类
public class Tiger extends Animal {
    @Override
    protected String introduce() {
        return "tiger";
    }

    @Override
    protected void shout() {
        System.out.println("I will eat you");
    }

    @Override
    protected void doWork() {
        System.out.println("I want to sleep");
    }
}
```

```java
// 实现类
public class Bird extends Animal {
    @Override
    protected String introduce() {
        return "bird";
    }

    @Override
    protected void shout() {
        System.out.println("ying ying ying");
    }

    @Override
    protected void doWork() {
        System.out.println("ying ying");
    }
}
```

```java
// 测试方法
public class Main {

    public static void main(String[] args) {
        Zoo zoo = new Zoo(new Bird());
        zoo.see();
        // 切换算法
        zoo.changeAnimal(new Tiger());
        zoo.see();
    }
}
```

```java
// 测试结果
bird discover you
ying ying ying
ying ying
tiger discover you
I will eat you
I want to sleep
```