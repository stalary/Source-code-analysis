
### 介绍
- 在Spring容器启动的过程中，会将Bean解析成Spring内部的BeanDefinition结构
- 用于进行注册/配置等操作

### 变量
```java
// scope，单例模式
String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;

// scope，非单例模式
String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;

// 应用
int ROLE_APPLICATION = 0;

// 复杂配置
int ROLE_SUPPORT = 1;

// 内部使用
int ROLE_INFRASTRUCTURE = 2;
```

### 方法
```java
// 设置当前Bean的父类名称
void setParentName(@Nullable String parentName);

// 返回父类名称
String getParentName();

// 设置Bean的className
void setBeanClassName(@Nullable String beanClassName);

// 获取的className
String getBeanClassName();

// 设置Bean的scope
void setScope(@Nullable String scope);

// 获取Bean的scope
String getScope();

// 设置Bean是否为懒加载
void setLazyInit(boolean lazyInit);

// 判断Bean是否为懒加载
boolean isLazyInit();

// 设置当前Bean依赖于哪些Bean
void setDependsOn(@Nullable String... dependsOn);

// 获取当前Bean的依赖
String[] getDependsOn();

// 设置是否可以注入到其他Bean中
void setAutowireCandidate(boolean autowireCandidate);

// 判断是否可以注入到其他Bean中
boolean isAutowireCandidate();

// 设置当前Bean是否为注入到其他类中的第一个
void setPrimary(boolean primary);

// 判断当前Bean是否为注入到其他类中的第一个
boolean isPrimary();

// 指定要使用的工厂Bean的名称
void setFactoryBeanName(@Nullable String factoryBeanName);

// 获取要使用的工厂Bean的名称
String getFactoryBeanName();

// 设置工厂方法的名称
void setFactoryMethodName(@Nullable String factoryMethodName);

// 获取工厂方法的名称
String getFactoryMethodName();

// 获取构造器的参数
ConstructorArgumentValues getConstructorArgumentValues();

// 判断是否有构造器参数
default boolean hasConstructorArgumentValues() {
		return !getConstructorArgumentValues().isEmpty();
}

// 获取Bean的属性
MutablePropertyValues getPropertyValues();

// 判断Bean是否存在属性
default boolean hasPropertyValues() {
		return !getPropertyValues().isEmpty();
}

// 判断Bean是否为单例
boolean isSingleton();

// 判断Bean是否为非单例
boolean isPrototype();

// 判断Bean是否是抽象的，即不能被实例化的
boolean isAbstract();

// 获取角色
int getRole();

// 获取Bean的描述
String getDescription();

// 获取资源的描述
String getResourceDescription();

// 获取当前Bean的BeanDefinition
BeanDefinition getOriginatingBeanDefinition();
```

