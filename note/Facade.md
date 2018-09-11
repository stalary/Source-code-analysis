- [介绍](#%E4%BB%8B%E7%BB%8D)
- [子系统](#%E5%AD%90%E7%B3%BB%E7%BB%9F)
- [门面类](#%E9%97%A8%E9%9D%A2%E7%B1%BB)
- [客户端](#%E5%AE%A2%E6%88%B7%E7%AB%AF)
### 介绍
- 属于结构型设计模式
- 优点：使子系统使用更加容易，实现子系统与用户直接的松耦合
- 缺点：未限制用户对子系统的使用，违背开闭原则，增加新的子系统需要修改外观类活着客户端的源码
- 门面模式应用在Tomcat的Request中

> 今天通过门面模式来描述一台电脑的启动和关闭
### 子系统
```java
public class CPU {
    public void start() {
        System.out.println("CPU start");
    }

    public void close() {
        System.out.println("CPU close");
    }
}
```

```java
public class Disk {
    public void start() {
        System.out.println("Disk start");
    }

    public void close() {
        System.out.println("Disk close");
    }
}
```

```java
public class Screen {
    public void start() {
        System.out.println("Screen start");
    }

    public void close() {
        System.out.println("Screen close");
    }
}
```

### 门面类
```java
public class Computer {

    private CPU cpu;

    private Disk disk;

    private Screen screen;

    public Computer() {
        this.cpu = new CPU();
        this.disk = new Disk();
        this.screen = new Screen();
    }

    public void start() {
        System.out.println("Computer start begin");
        cpu.start();
        disk.start();
        screen.start();
        System.out.println("Computer start finish");
    }

    public void close() {
        System.out.println("Computer close begin");
        cpu.close();
        disk.close();
        screen.close();
        System.out.println("Computer close finish");
    }
}
```

### 客户端
```java
public class Main {

    public static void main(String[] args) {
        Computer computer = new Computer();
        computer.start();
        System.out.println("use computer doing something...");
        computer.close();
    }
}
```

```java
// 结果
Computer start begin
CPU start
Disk start
Screen start
Computer start finish
use computer doing something...
Computer close begin
CPU close
Disk close
Screen close
Computer close finish

```
