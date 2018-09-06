- [介绍](#介绍)
- [常量&变量](#常量变量)
- [构造](#构造)
- [get](#get)
- [put](#put)

### 介绍

- 继承AbstractMap，实现了Serializable和Cloneable，为Enum类型打造的Map实现。
- 非线程安全
- 内部元素的顺序为自然顺序，也就是被声明的顺序。
- key必须是Enum，不能为null，但value可以是null


### 常量&变量

```java
// 类型为Enum的key的Class对象
private final Class<K> keyType;

// 缓存该Enum中所有的成员
private transient K[] keyUniverse;

/**
 * Array representation of this map.  The ith element is the value
 * to which universe[i] is currently mapped, or null if it isn't
 * mapped to anything, or NULL if it's mapped to null.
 */
// 代表这个map的数组。null代表什么也不映射，NULL代表映射null
private transient Object[] vals;

// 代表null的匿名类
private static final Object NULL = new Object() {
    public int hashCode() {
        return 0;
    }

    public String toString() {
        return "java.util.EnumMap.NULL";
    }
};

// map中映射数
private transient int size = 0;

// 一个entryset，第一次请求的时候才会初始化
private transient Set<Map.Entry<K,V>> entrySet;


```

### 构造

```java

// 传入目标类型的class对象
public EnumMap(Class<K> keyType) {
    this.keyType = keyType;
    keyUniverse = getKeyUniverse(keyType);
    // 这里可以看到，vals数组的长度就是enum中成员的个数
    vals = new Object[keyUniverse.length];
}

// 传入一个EnumMap，和原来的一样
public EnumMap(EnumMap<K, ? extends V> m) {
        keyType = m.keyType;
        keyUniverse = m.keyUniverse;
        vals = m.vals.clone();
        size = m.size;
    }
}

// 传入Map
public EnumMap(Map<K, ? extends V> m) {
    if (m instanceof EnumMap) {
        // 按照上面的方法实现
        EnumMap<K, ? extends V> em = (EnumMap<K, ? extends V>) m;
        keyType = em.keyType;
        keyUniverse = em.keyUniverse;
        vals = em.vals.clone();
        size = em.size;
    } else {
        if (m.isEmpty())
            throw new IllegalArgumentException("Specified map is empty");
        // 取出一个key获得他的类型
        keyType = m.keySet().iterator().next().getDeclaringClass();
        keyUniverse = getKeyUniverse(keyType);
        vals = new Object[keyUniverse.length];
        // 全部放入
        putAll(m);
    }
}

```

### get

```java
// 检验key的合法性，然后unmaskNull后返回
public V get(Object key) {
    return (isValidKey(key) ?
            unmaskNull(vals[((Enum<?>)key).ordinal()]) : null);
}
private boolean isValidKey(Object key) {
    if (key == null)
        return false;

    // 比instanceof Enum效率高
    Class<?> keyClass = key.getClass();
    return keyClass == keyType || keyClass.getSuperclass() == keyType;
}

private V unmaskNull(Object value) {
    // 如果返回是NULL，转成null
    return (V)(value == NULL ? null : value);
}

```


### put

```java

public V put(K key, V value) {
    typeCheck(key);
    // 获取声明顺序，放入vals对应位置
    int index = key.ordinal();
    Object oldValue = vals[index];
    // 如果是null，映射为NULL对象
    vals[index] = maskNull(value)
    ;
    // 如果是新增的映射，size++
    if (oldValue == null)
        size++;
    return unmaskNull(oldValue);
}
// 检出非法的类型
private void typeCheck(K key) {
    Class<?> keyClass = key.getClass();
    if (keyClass != keyType && keyClass.getSuperclass() != keyType)
        throw new ClassCastException(keyClass + " != " + keyType);
}
//  转化null为NULL
private Object maskNull(Object value) {
    return (value == null ? NULL : value);
}

```

