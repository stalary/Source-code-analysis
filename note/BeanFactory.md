- [介绍](#%E4%BB%8B%E7%BB%8D)
- [方法](#%E6%96%B9%E6%B3%95)
### 介绍
- 用于解决Bean直接的依赖问题
- 方便对Bean进行处理，实现单例或者其他类型的Bean 

### 方法
```java
// 通过Bean的名称获取Bean，返回的类型为Object
Object getBean(String name) throws BeansException;

// 通过Bean的名称和类型获取Bean，获取的类型为指定的类型
<T> T getBean(String name, @Nullable Class<T> requiredType) throws BeansException;

// 显示指定参数来通过特定的构造函数获取Bean
Object getBean(String name, Object... args) throws BeansException;

// 直接通过类型来获取Bean
<T> T getBean(Class<T> requiredType) throws BeansException;

// 通过类型和参数通过特定的构造函数获取Bean
<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

// 判断BeanFactory中是否包含指定名称的Bean
boolean containsBean(String name);

// 判断指定名称的Bean是否是单例的
boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

// 判断指定名称的Bean是否是原型的(每次获取Bean都新建一个)
boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

// 判断指定名称的Bean是否是指定类型的
boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

boolean isTypeMatch(String name, @Nullable Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

// 获取指定名称的Bean的类型
Class<?> getType(String name) throws NoSuchBeanDefinitionException;

// 获取指定名称的Bean的别名
String[] getAliases(String name);
```