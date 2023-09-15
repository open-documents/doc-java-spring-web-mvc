

# 一、初始化代码

```java
/* ------------------------------------ DispatcherServlet ------------------------------------
 */
private void initHandlerMappings(ApplicationContext context) {  
	// List<HandlerMapping> handlerMappings;
    this.handlerMappings = null;  

	// 1. 从DispatcherServlet所属的WebApplicationContext及其父WebApplicationContext中寻找HandlerMapping实现类
	// 2. 

	//  this.detectAllHandlerMappings默认(初)值为true
    if (this.detectAllHandlerMappings) {  
        // 
        Map<String, HandlerMapping> matchingBeans =  
            BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);  
        if (!matchingBeans.isEmpty()) {  
            this.handlerMappings = new ArrayList<>(matchingBeans.values());  
            // We keep HandlerMappings in sorted order.  
            AnnotationAwareOrderComparator.sort(this.handlerMappings);  
        }  
    }  
    else {
        try {  
	        // HANDLER_MAPPING_BEAN_NAME = "handlerMapping"
            HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);  
            this.handlerMappings = Collections.singletonList(hm);  
        }  
        ... // catch
    }  
  
	// 确保至少有一个HandlerMapping实现类
    if (this.handlerMappings == null) {  

	    this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);  
		... // log
    }  
  
    for (HandlerMapping mapping : this.handlerMappings) {  
        if (mapping.usesPathPatterns()) {  
            this.parseRequestPath = true;  
            break;      
		}  
    }  
}

```

# 二、获取默认组件
```java
/* ------------------------------------ DispatcherServlet ------------------------------------
 */
protected <T> List<T> getDefaultStrategies(ApplicationContext context, Class<T> strategyInterface) {  

    if (defaultStrategies == null) {  
        try {  
	        // DEFAULT_STRATEGIES_PATH = "DispatcherServlet.properties"
			ClassPathResource resource = 
				new ClassPathResource(DEFAULT_STRATEGIES_PATH, DispatcherServlet.class);  
            defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);  
        }  
		... // catch
    }  
  
    String key = strategyInterface.getName();  
    String value = defaultStrategies.getProperty(key);  
    if (value != null) {  
	    // 1. 将被逗号分割的String转换为String[]
    	// 2. 默认下面3个HandlerMapping实现类:
		//    - org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,  
	    //    - org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping
		//    - org.springframework.web.servlet.function.support.RouterFunctionMapping
        String[] classNames = StringUtils.commaDelimitedListToStringArray(value);  
        List<T> strategies = new ArrayList<>(classNames.length);  
        for (String className : classNames) {  
	        try {  
	            Class<?> clazz = ClassUtils.forName(className, DispatcherServlet.class.getClassLoader());  
	            Object strategy = createDefaultStrategy(context, clazz);  
	            strategies.add((T) strategy);  
            }  
			... // catch
        }  
        return strategies;  
    }  
    else {  
        return Collections.emptyList();  
    }  
}

/* ------------------------------------ DispatcherServlet ------------------------------------
 */
protected Object createDefaultStrategy(ApplicationContext context, Class<?> clazz) {  
    return context.getAutowireCapableBeanFactory().createBean(clazz);  
}

```

# 二、总结

