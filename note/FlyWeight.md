- [介绍](#%E4%BB%8B%E7%BB%8D)
- [抽象角色](#%E6%8A%BD%E8%B1%A1%E8%A7%92%E8%89%B2)
- [具体角色](#%E5%85%B7%E4%BD%93%E8%A7%92%E8%89%B2)
- [享元工厂](#%E4%BA%AB%E5%85%83%E5%B7%A5%E5%8E%82)
- [客户端](#%E5%AE%A2%E6%88%B7%E7%AB%AF)
### 介绍
- 属于结构型设计模式
- 优点：外部状态(不可共享)相对独立，使得对象可以适应不同的外部环境，可共享相同的对象，减少内存消耗
- 缺点：外部状态由客户端保存，要求内外部状态分离，使得程序的逻辑复杂化
- 与单例模式区别是，系统中存在多个相同的对象，共享一个对象的拷贝，可看作单例模式的扩展
- 享元模式应用在数据库连接池
  
### 抽象角色
```java
public interface Person {

    void introduce(String info);
}
```

### 具体角色
```java
public class PersonImpl implements Person {

    private String name;

    public PersonImpl(String name) {
        this.name = name;
    }

    @Override
    public void introduce(String info) {
        System.out.println("inner parameter:" + this.name);
        System.out.println("outer parameter:" + info);
    }
}
```

### 享元工厂
```java
public class FlyWeightFactory {

    private Map<String, Person> map = new HashMap<>();

    // 获取指定对象
    public Person getPerson(String name) {
        Person person = map.get(name);
        if (person == null) {
            person = new PersonImpl(name);
            map.put(name, person);
        }
        return person;
    }

    // 获取当前对象数量
    public int getCount() {
        return map.size();
    }
}
```

### 客户端
```java
public class Main {

    public static void main(String[] args) {
        FlyWeightFactory factory = new FlyWeightFactory();
        Person person = factory.getPerson("stalary");
        person.introduce("第一次获取");
        person = factory.getPerson("hawk");
        person.introduce("第二次获取");
        person = factory.getPerson("stalary");
        person.introduce("第三次获取");
        System.out.println("当前对象数量:" + factory.getCount());
    }
}
```

```java
// 测试结果
inner parameter:stalary
outer parameter:第一次获取
inner parameter:hawk
outer parameter:第二次获取
inner parameter:stalary
outer parameter:第三次获取
当前对象数量:2
```