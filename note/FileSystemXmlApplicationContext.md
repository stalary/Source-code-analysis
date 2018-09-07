- [介绍](#%E4%BB%8B%E7%BB%8D)
- [具体方法](#%E5%85%B7%E4%BD%93%E6%96%B9%E6%B3%95)
### 介绍
- 对ApplicationContext的标准实现
- 包含了容器的启动，IOC的初始化

### 具体方法
```java
public class FileSystemXmlApplicationContext extends AbstractXmlApplicationContext {
    // 传入ApplicationContext，直接调用父类构造方法
    public FileSystemXmlApplicationContext(ApplicationContext parent) {
		super(parent);
	}
    // 传入xml配置文件路径
	public FileSystemXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}

    // 传入多个xml配置文件路径
    public FileSystemXmlApplicationContext(String... configLocations) throws BeansException {
        // 传入到接收数组参数的构造方法中
		this(configLocations, true, null);
	}

    // 传入xml配置文件路径数组和ApplicationContext
    public FileSystemXmlApplicationContext(String[] configLocations, ApplicationContext parent) throws BeansException {
		this(configLocations, true, parent);
	}

    // 传入xml配置文件路径数组和是否刷新
    public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh) throws BeansException {
		this(configLocations, refresh, null);
	}

    // 传入配置文件数组，是否刷新，以及ApplicationContext
    public FileSystemXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {
        // 首先直接调用父类构造函数
		super(parent);
        // 设置配置
		setConfigLocations(configLocations);
        // 当需要刷新时进行刷新操作
		if (refresh) {
			refresh();
		}
	}

    // 通过路径获取Resource
    @Override
	protected Resource getResourceByPath(String path) {
        // 对传入的path进行处理
		if (path.startsWith("/")) {
			path = path.substring(1);
		}
		return new FileSystemResource(path);
	}
}
```

- 可以看出FileSystemXmlApplicationContext中只有一些构造方法，而核心的方法其实是AbstractApplicationContext中的refresh()