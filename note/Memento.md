### 介绍
- 属于行为型设计模式
- 优点：保存备份，可以遇到错误时进行还原，同时备份状态保存在发起人角色之外，发起人角色不需要对各个备份的状态进行管理
- 缺点：资源消耗比较严重

### 角色
```java
public class Role {

    /**
     * 提交标签
     */
    private String tag;

    /**
     * 提交时间
     */
    private LocalDateTime time;

    /**
     * 提交代码内容
     */
    private String code;

    public String getTag() {
        return tag;
    }

    public void setTag(String tag) {
        this.tag = tag;
    }

    public LocalDateTime getTime() {
        return time;
    }

    public void setTime(LocalDateTime time) {
        this.time = time;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public Role(String tag, LocalDateTime time, String code) {
        this.tag = tag;
        this.time = time;
        this.code = code;
    }

    public Bean setBean() {
        return new Bean(this.tag, this.time, this.code);
    }

    public void recover(Bean bean) {
        this.tag = bean.getTag();
        this.time = bean.getTime();
        this.code = bean.getCode();
    }

    @Override
    public String toString() {
        return "Role{" +
                "tag='" + tag + '\'' +
                ", time=" + time +
                ", code='" + code + '\'' +
                '}';
    }
}
```

### 状态存储
```java
public class Bean {

    /**
     * 提交标签
     */
    private String tag;

    /**
     * 提交时间
     */
    private LocalDateTime time;

    /**
     * 提交代码内容
     */
    private String code;

    public String getTag() {
        return tag;
    }

    public LocalDateTime getTime() {
        return time;
    }

    public String getCode() {
        return code;
    }

    public Bean(String tag, LocalDateTime time, String code) {
        this.tag = tag;
        this.time = time;
        this.code = code;
    }
}
```

### 角色管理
```java
public class Manager {

    private Bean bean;

    public Bean getBean() {
        return bean;
    }

    public void setBean(Bean bean) {
        this.bean = bean;
    }
}
```

### 客户端
```java
public class Main {

    public static void main(String[] args) {
        Role role = new Role("1.0", LocalDateTime.now(), "hello world");
        System.out.println("修改前：" + role);
        Manager manager = new Manager();
        manager.setBean(role.setBean());
        role.setCode("test update");
        role.setTag("1.1");
        role.setTime(LocalDateTime.now());
        System.out.println("修改后：" + role);
        role.recover(manager.getBean());
        System.out.println("回滚后：" + role);
    }
}
```

### 测试结果
```java
修改前：Role{tag='1.0', time=2018-09-14T10:29:24.270, code='hello world'}
修改后：Role{tag='1.1', time=2018-09-14T10:29:24.274, code='test update'}
回滚后：Role{tag='1.0', time=2018-09-14T10:29:24.270, code='hello world'}
```