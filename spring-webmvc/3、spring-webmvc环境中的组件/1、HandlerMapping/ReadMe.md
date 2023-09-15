
# 总体关系继承图

![[Pasted image 20230623035959.png]]

```java
/* -------------------------------- DispatcherServlet --------------------------------

 */
protected <T> List<T> getDefaultStrategies(ApplicationContext context, Class<T> strategyInterface) {  
	// this.defaultStrategies(Properties) 默认为null
    if (defaultStrategies == null) {  
        try {  
	        // DEFAULT_STRATEGIES_PATH = "DispatcherServlet.properties"
            ClassPathResource resource = new ClassPathResource(DEFAULT_STRATEGIES_PATH, DispatcherServlet.class);  
            defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);  
        }  
		... // catch
    }  
  
    String key = strategyInterface.getName();  
    String value = defaultStrategies.getProperty(key);  
    if (value != null) {  
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
/* -------------------------------- DispatcherServlet --------------------------------

 */
protected Object createDefaultStrategy(ApplicationContext context, Class<?> clazz) {  
    return context.getAutowireCapableBeanFactory().createBean(clazz);  
}
```