- [介绍](#%E4%BB%8B%E7%BB%8D)
- [原型类](#%E5%8E%9F%E5%9E%8B%E7%B1%BB)
- [客户端](#%E5%AE%A2%E6%88%B7%E7%AB%AF)
### 介绍
- 创建型设计模式
- 与克隆相同
- 可以避免实例化时的内存消耗

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