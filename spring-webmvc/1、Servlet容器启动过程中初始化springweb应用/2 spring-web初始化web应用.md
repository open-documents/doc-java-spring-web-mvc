
# 初始化容器

知道了spring-web提供的SpringServletContainerInitializer类提供了初始化容器的过程，那么先具体看初始化容器的代码。
```java
public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)  
      throws ServletException {  
    List<WebApplicationInitializer> initializers = Collections.emptyList();  

	// 对满足@HandlesTypes指定类的条件的类作筛选,以排除恶意的不满足条件的类
	if (webAppInitializerClasses != null) {  
        initializers = new ArrayList<>(webAppInitializerClasses.size());  
		for (Class<?> waiClass : webAppInitializerClasses) {  
            // Be defensive: Some servlet containers provide us with invalid classes,  
		    // no matter what @HandlesTypes says...         
		    if (!waiClass.isInterface() 
			    && !Modifier.isAbstract(waiClass.getModifiers()) 
			    && WebApplicationInitializer.class.isAssignableFrom(waiClass)) {  
	            try {  
		            // 实例化满足条件的类
	                initializers.add( (WebApplicationInitializer)
				        ReflectionUtils.accessibleConstructor(waiClass).newInstance());  
                }  
	            catch (Throwable ex) {  
	                throw new ServletException("Failed to instantiate WebApplicationInitializer class", ex);  
                }  
            }  
        }  
    }  
  
    if (initializers.isEmpty()) {  
        servletContext.log("No Spring WebApplicationInitializer types detected on classpath");  
        return; 
	}  
  
    servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");  
    AnnotationAwareOrderComparator.sort(initializers);  
    // 调用WebApplicationInitializer来初始化Web应用所需要的前置环境
    for (WebApplicationInitializer initializer : initializers) {  
        initializer.onStartup(servletContext);  
    }  
}
```


