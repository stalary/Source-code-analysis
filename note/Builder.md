- [介绍](#%E4%BB%8B%E7%BB%8D)
- [建造者](#%E5%BB%BA%E9%80%A0%E8%80%85)
- [测试类](#%E6%B5%8B%E8%AF%95%E7%B1%BB)
### 介绍
- 属于创建型设计模式
- 优点：建造与源代码分离
- 缺点：增加代码量
- 建造者模式应用在StringBuiler和StringBuffer的append方法

> 今天通过建造者方法来进行创建对象
### 建造者
```java
public class Person {

    private final String name;

    private final int age;

    private final String sex;

    private final String address;

    private Person(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.sex = builder.sex;
        this.address = builder.address;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public String getSex() {
        return sex;
    }

    public String getAddress() {
        return address;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }

    public static class Builder {

        private final String name;

        private final String sex;

        private int age;

        private String address;

        public Builder(String name, String sex) {
            // 输入校验
            if (name == null || sex == null) {
                throw new IllegalArgumentException("name and sex can not be null");
            }
            if (!sex.equals("man") && !sex.equals("woman")) {
                throw new IllegalArgumentException("sex input error!");
            }
            this.name = name;
            this.sex = sex;
        }

        public Builder withAge(int age) {
            this.age = age;
            return this;
        }

        public Builder withAddress(String address) {
            this.address = address;
            return this;
        }

        public Person build() {
            return new Person(this);
        }
    }
}
```

### 测试类
```java
public class Main {

    public static void main(String[] args) {
        Person hawk = new Person.Builder("hawk", "man")
                .withAddress("jinan")
                .withAge(22)
                .build();
        System.out.println(hawk);
        Person claire = new Person.Builder("claire", "woman")
                .withAddress("beijing")
                .withAge(21)
                .build();
        System.out.println(claire);
        Person stalary = new Person.Builder("stalary", "man1")
                .withAddress("beijing")
                .withAge(22)
                .build();
        System.out.println(stalary);
    }
}
```

```java
// 测试结果
Exception in thread "main" java.lang.IllegalArgumentException: sex input error!
	at com.stalary.designpattern.builder.Person$Builder.<init>(Person.java:72)
	at com.stalary.designpattern.builder.Main.main(Main.java:27)
Person{name='hawk', age=22, sex='man', address='jinan'}
Person{name='claire', age=21, sex='woman', address='beijing'}
```
