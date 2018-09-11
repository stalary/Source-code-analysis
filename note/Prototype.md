- [介绍](#%E4%BB%8B%E7%BB%8D)
- [原型类](#%E5%8E%9F%E5%9E%8B%E7%B1%BB)
- [客户端](#%E5%AE%A2%E6%88%B7%E7%AB%AF)
### 介绍
- 属于创建型设计模式
- 优点：简化对象的的创建过程，扩展性好，可以使用深克隆保存对象的状态
- 缺点：需要为每一个类配置一个克隆方法，并且位于类的内部，改造类时需要修改代码，违反开闭原则
- 适用场景：创建新对象成本较大，需要保存对象状态
- 原型模式应用在 Spring Bean 的创建中

> 今天通过原型方法来实现快递的批量寄送
### 原型类
```java
// 提供构造函数，传入无需改变的模版参数
// 提供set方法，对需要变化的参数进行赋值
public class Express implements Cloneable {

    private String receiver;

    private String sender;

    private String goods;

    private String receiveAddress;

    private String sendAddress;

    public Express(String sender, String sendAddress) {
        this.sender = sender;
        this.sendAddress = sendAddress;
    }

    public void setReceiver(String receiver) {
        this.receiver = receiver;
    }

    public void setGoods(String goods) {
        this.goods = goods;
    }

    public void setReceiveAddress(String receiveAddress) {
        this.receiveAddress = receiveAddress;
    }

    @Override
    protected Express clone() {
        try {
            return (Express) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    public String toString() {
        return "Express{" +
                "receiver='" + receiver + '\'' +
                ", sender='" + sender + '\'' +
                ", goods='" + goods + '\'' +
                ", receiveAddress='" + receiveAddress + '\'' +
                ", sendAddress='" + sendAddress + '\'' +
                '}';
    }
}
```

### 客户端
```java
// 测试方法
public class Main {

    public static void main(String[] args) {
        Express express = new Express("stalary", "beijing");
        Map<Integer, List<String>> data = new HashMap<>();
        data.put(1, Lists.newArrayList("hawk", "jinan"));
        data.put(2, Lists.newArrayList("claire", "beijing"));
        data.put(3, Lists.newArrayList("none", "none"));
        for (int i = 1; i <= 3; i ++) {
            express.clone();
            express.setGoods("good:" + i);
            express.setReceiver(data.get(i).get(0));
            express.setReceiveAddress(data.get(i).get(1));
            System.out.println(express);
        }
    }
}
```

```java
// 测试结果
Express{receiver='hawk', sender='stalary', goods='good:1', receiveAddress='jinan', sendAddress='beijing'}
Express{receiver='claire', sender='stalary', goods='good:2', receiveAddress='beijing', sendAddress='beijing'}
Express{receiver='none', sender='stalary', goods='good:3', receiveAddress='none', sendAddress='beijing'}
```