### 介绍
- 用于解析 @RequestMapping 

### afterPropertiesSet
设置一些前置配置
```java
public void afterPropertiesSet() {
		this.config = new RequestMappingInfo.BuilderConfiguration();
		this.config.setUrlPathHelper(getUrlPathHelper());
		this.config.setPathMatcher(getPathMatcher());
        this.config.setSuffixPatternMatch(this.useSuffixPatternMatch); // 使用前缀模式匹配
		this.config.setTrailingSlashMatch(this.useTrailingSlashMatch); // 兼容末尾的/
		this.config.setRegisteredSuffixPatternMatch(this.useRegisteredSuffixPatternMatch);
		this.config.setContentNegotiationManager(getContentNegotiationManager());

		super.afterPropertiesSet();
}
```

### isHandler
判断是否需要进行处理
```java
        // 当存在Controller或者RequestMapping注解时才进行处理
	protected boolean isHandler(Class<?> beanType) {
		return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
				AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
	}
```

### getMappingForMethod
获取方法的RequestMapping信息
```java
	protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
        // 创建方法的映射信息
		RequestMappingInfo info = createRequestMappingInfo(method);
		if (info != null) {
            // 合并方法和类型的映射信息
			RequestMappingInfo typeInfo = createRequestMappingInfo(handlerType);
			if (typeInfo != null) {
				info = typeInfo.combine(info);
			}
		}
		return info;
	}
```