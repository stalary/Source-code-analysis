- [介绍](#%E4%BB%8B%E7%BB%8D)
- [静态代理](#%E9%9D%99%E6%80%81%E4%BB%A3%E7%90%86)
- [jdk动态代理](#jdk%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86)
- [cglib动态代理](#cglib%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86)
### 介绍
- 属于结构型设计模式
- 分为三种：1.静态代理 2.动态代理 3.cglib代理
- 提供了对目标对象间接访问的方式
- 可在原对象基础上进行功能的扩展

### 静态代理
静态代理需要定义接口，然后进行实现
```java
// 接口
public interface Zoo {

    void enter(User user);
}
```

```java
// 使用对象
@Data
@AllArgsConstructor
public class User {

    private Integer id;

    private String username;
}
```

```java
// 目标对象
public class ZooImpl implements Zoo {
    @Override
    public void enter(User user) {
        System.out.println(user + " enter ZooImpl");
    }
}
```

```java
// 代理类
public class ZooProxy implements Zoo {

    private Integer num = 1;

    private static final Integer MAX_NUM = 2;

    /** 目标对象 **/
    private Zoo zoo;

    public ZooProxy(Zoo zoo) {
        this.zoo = zoo;
    }

    @Override
    public void enter(User user) {
        if (num <= MAX_NUM) {
            // 执行之前
            System.out.println(user + " can enter");
            num++;
            zoo.enter(user);
            // 执行之后
            System.out.println(user + " left");
        } else {
            System.out.println(user + " is not allowed to enter");
        }
    }
}
```

```java
// 测试类
public class Main {

    public static void main(String[] args) {
        ZooProxy proxy = new ZooProxy(new ZooImpl());
        proxy.enter(new User(1, "stalary"));
        proxy.enter(new User(2, "hawk"));
        proxy.enter(new User(3, "claire"));
    }
}
```

```java
// 测试结果
User(id=1, username=stalary) can enter
User(id=1, username=stalary) enter ZooImpl
User(id=1, username=stalary) left
User(id=2, username=hawk) can enter
User(id=2, username=hawk) enter ZooImpl
User(id=2, username=hawk) left
User(id=3, username=claire) is not allowed to enter
```

### jdk动态代理
动态代理不需要实现接口，但是需要指定接口类型，即目标对象需要实现接口
```java
// ClassLoader loader：目标对象使用的类加载器
// Class<?>[] interfaces：目标对象实现的接口类型
// InvocationHandler：事件处理方法
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        // 检验处理方法不为空
        Objects.requireNonNull(h);
        // 将传入的接口克隆
        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            // 安全检查
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        // 查找或生成指定的代理类
        Class<?> cl = getProxyClass0(loader, intfs);

        try {
            if (sm != null) {
                // 安全检查
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
            // 获取处理器的构造函数
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            // 当构造器不可修改时，修改权限
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            // 返回实例
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```

```java
// 代理工厂
public class ProxyFactory {

    // 目标对象
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    public Object getProxyInstance() {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                (proxy, method, args) -> {
                    System.out.println("动态代理开始执行");
                    Object value = method.invoke(target, args);
                    System.out.println("动态代理执行完毕");
                    return value;
                }
        );
    }
}
```

```java
// 测试类
public class Main1 {

    public static void main(String[] args) {
        Zoo zoo = new ZooImpl();
        Zoo proxy = (Zoo) new ProxyFactory(zoo).getProxyInstance();
        proxy.enter(new User(1, "stalary"));
        proxy.enter(new User(2, "hawk"));
        proxy.enter(new User(3, "claire"));
    }
}
```

```java
// 结果
动态代理开始执行
User(id=1, username=stalary) enter ZooImpl
动态代理执行完毕
动态代理开始执行
User(id=2, username=hawk) enter ZooImpl
动态代理执行完毕
动态代理开始执行
User(id=3, username=claire) enter ZooImpl
动态代理执行完毕
```

### cglib动态代理
可使用在目标对象未使用接口时，但是需要依赖于cglib的jar包
```java
// 代理类，实现MethodInterceptor
public class ProxyCGLIB implements MethodInterceptor {

    private Object target;

    public ProxyCGLIB(Object target) {
        this.target = target;
    }

    public Object getProxyInstance() {
        //1.工具类
        Enhancer en = new Enhancer();
        //2.设置父类
        en.setSuperclass(target.getClass());
        //3.设置回调函数
        en.setCallback(this);
        //4.创建子类(代理对象)
        return en.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("cglib开始执行");
        Object value = method.invoke(target, args);
        System.out.println("cglib执行完毕");
        return value;
    }
}
```

```java
// 目标对象
public class NormalZoo {

    public void enter(User user) {
        System.out.println(user + " enter NormalZoo");
    }
}
```

```java
// 测试方法
public class Main2 {

    public static void main(String[] args) {
        NormalZoo target = new NormalZoo();
        NormalZoo proxy = (NormalZoo) new ProxyCGLIB(target).getProxyInstance();
        proxy.enter(new User(1, "stalary"));
        proxy.enter(new User(2, "hawk"));
        proxy.enter(new User(3, "claire"));
    }
}
```

```java
// 测试结果
cglib开始执行
User(id=1, username=stalary) enter NormalZoo
cglib执行完毕
cglib开始执行
User(id=2, username=hawk) enter NormalZoo
cglib执行完毕
cglib开始执行
User(id=3, username=claire) enter NormalZoo
cglib执行完毕
```