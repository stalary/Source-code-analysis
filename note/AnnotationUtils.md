- [介绍](#%E4%BB%8B%E7%BB%8D)
- [方法](#%E6%96%B9%E6%B3%95)
    - [getAnnotation(Spring 3.1出现，推荐使用findAnnotation)](#getannotationspring-31%E5%87%BA%E7%8E%B0%E6%8E%A8%E8%8D%90%E4%BD%BF%E7%94%A8findannotation)
    - [getAnnotations](#getannotations)
    - [getRepeatableAnnotations](#getrepeatableannotations)
    - [findAnnotation(Spring 4.2出现，避免了无限递归的问题)](#findannotationspring-42%E5%87%BA%E7%8E%B0%E9%81%BF%E5%85%8D%E4%BA%86%E6%97%A0%E9%99%90%E9%80%92%E5%BD%92%E7%9A%84%E9%97%AE%E9%A2%98)
    - [getValue](#getvalue)
    - [synthesizeAnnotation](#synthesizeannotation)
    - [isAnnotationDeclaredLocally](#isannotationdeclaredlocally)
    - [isAnnotationInherited](#isannotationinherited)
### 介绍
- Spring 提供的注解工具类

### 方法
#### getAnnotation(Spring 3.1出现，推荐使用findAnnotation)
```java
// 查找指定的注解，不存在时返回null
public static <A extends Annotation> A getAnnotation(Annotation annotation, Class<A> annotationType) {
                // 当注解是指定类型的一个实例时，返回合成后的注解
		if (annotationType.isInstance(annotation)) {
			return synthesizeAnnotation((A) annotation);
		}
                // 获取注解的类型
		Class<? extends Annotation> annotatedElement = annotation.annotationType();
		try {
            // 获取指定类型的元注解，也可以直接传入AnnotatedElement
			A metaAnn = annotatedElement.getAnnotation(annotationType);
            // 当元注解存在时返回合成后的注解，否则返回null
			return (metaAnn != null ? synthesizeAnnotation(metaAnn, annotatedElement) : null);
		}
		catch (Throwable ex) {
			handleIntrospectionFailure(annotatedElement, ex);
			return null;
		}
}
```

```java
// 查找方法中指定类型的注解
public static <A extends Annotation> A getAnnotation(Method method, Class<A> annotationType) {
                // 转化为可转化为注解元素的方法
		Method resolvedMethod = BridgeMethodResolver.findBridgedMethod(method);
		return getAnnotation((AnnotatedElement) resolvedMethod, annotationType);
}
```

#### getAnnotations
```java
// 获取注解元素上包含的注解
public static Annotation[] getAnnotations(AnnotatedElement annotatedElement) {
		try {
            // 返回合成注解的数组(也可以传入方法进行转化)
			return synthesizeAnnotationArray(annotatedElement.getAnnotations(), annotatedElement);
		}
		catch (Throwable ex) {
			handleIntrospectionFailure(annotatedElement, ex);
			return null;
		}
	}
```

#### getRepeatableAnnotations
```java
// 获取重复注解(@Repeatable)
private static <A extends Annotation> Set<A> getRepeatableAnnotations(AnnotatedElement annotatedElement,
			Class<A> annotationType, @Nullable Class<? extends Annotation> containerAnnotationType, boolean declaredMode) {

		try {
                        // 当传入的注解元素为方法时，需要进行转化
			if (annotatedElement instanceof Method) {
				annotatedElement = BridgeMethodResolver.findBridgedMethod((Method) annotatedElement);
			}
                        // 返回注解的集合
			return new AnnotationCollector<>(annotationType, containerAnnotationType, declaredMode).getResult(annotatedElement);
		}
		catch (Throwable ex) {
			handleIntrospectionFailure(annotatedElement, ex);
			return Collections.emptySet();
		}
}
```

#### findAnnotation(Spring 4.2出现，避免了无限递归的问题)
```java
private static <A extends Annotation> A findAnnotation(
			AnnotatedElement annotatedElement, Class<A> annotationType, Set<Annotation> visited) {
		try {
                        // 获取注解元素上的注解
			A annotation = annotatedElement.getDeclaredAnnotation(annotationType);
                        // 存在注解时直接返回
			if (annotation != null) {
				return annotation;
			}
                        // 否则查找注解元素上的所有注解
			for (Annotation declaredAnn : annotatedElement.getDeclaredAnnotations()) {
				Class<? extends Annotation> 
                // 获取单个注解元素上的注解类型 
                declaredType = declaredAnn.annotationType();
                // 当不是java包中的注解，并且set中不存在该注解时，递归查找子注解
				if (!isInJavaLangAnnotationPackage(declaredType) && visited.add(declaredAnn)) {
					annotation = findAnnotation((AnnotatedElement) declaredType, annotationType, visited);
					if (annotation != null) {
						return annotation;
					}
				}
			}
		}
		catch (Throwable ex) {
			handleIntrospectionFailure(annotatedElement, ex);
		}
		return null;
	}
```

#### getValue
```java
// 获取注解指定属性的值
public static Object getValue(@Nullable Annotation annotation, @Nullable String attributeName) {
                // 当注解为null或者不存在指定属性时，返回null 
		if (annotation == null || !StringUtils.hasText(attributeName)) {
			return null;
		}
		try {
            // 获取指定类型的方法
			Method method = annotation.annotationType().getDeclaredMethod(attributeName);
            // 设置权限为可写
			ReflectionUtils.makeAccessible(method);
            // 调用
			return method.invoke(annotation);
		}
		catch (NoSuchMethodException ex) {
			return null;
		}
		catch (InvocationTargetException ex) {
			rethrowAnnotationConfigurationException(ex.getTargetException());
			throw new IllegalStateException(
					"Could not obtain value for annotation attribute '" + attributeName + "' in " + annotation, ex);
		}
		catch (Throwable ex) {
			handleIntrospectionFailure(annotation.getClass(), ex);
			return null;
		}
}
```

#### synthesizeAnnotation
```java
// 通过动态代理将参数合成到指定的注解元素上的注解中
public static <A extends Annotation> A synthesizeAnnotation(Map<String, Object> attributes,
			Class<A> annotationType, @Nullable AnnotatedElement annotatedElement) {
// 获取注解参数处理器
		MapAnnotationAttributeExtractor attributeExtractor =
				new MapAnnotationAttributeExtractor(attributes, annotationType, annotatedElement);
                // 获得合成处理器
		InvocationHandler handler = new SynthesizedAnnotationInvocationHandler(attributeExtractor);
		Class<?>[] exposedInterfaces = (canExposeSynthesizedMarker(annotationType) ?
				new Class<?>[] {annotationType, SynthesizedAnnotation.class} : new Class<?>[] {annotationType});
        // 使用动态代理写入参数
		return (A) Proxy.newProxyInstance(annotationType.getClassLoader(), exposedInterfaces, handler);
}
```

#### isAnnotationDeclaredLocally
```java
// 判断类中是否存在指定的注解，并且不是继承来的
public static boolean isAnnotationDeclaredLocally(Class<? extends Annotation> annotationType, Class<?> clazz) {
		try {
			return (clazz.getDeclaredAnnotation(annotationType) != null);
		}
		catch (Throwable ex) {
			handleIntrospectionFailure(clazz, ex);
			return false;
		}
}
```

#### isAnnotationInherited
```java
// 判断类中是否存在指定的注解，并且是继承来的
public static boolean isAnnotationInherited(Class<? extends Annotation> annotationType, Class<?> clazz) {
		return (clazz.isAnnotationPresent(annotationType) && !isAnnotationDeclaredLocally(annotationType, clazz));
}
```

