

# 方法--初始化WebApplicationContext

WebApplicationContext来源从上到下依次是：
- 创建FrameworkServlet实例时，通过构造函数指定。
- 尝试从ServletContext中获取 `this.contextAttribute`键 对应的 WebApplicationContext对象。
- 创建 `this.contextClass` 对应的WebApplicationContext实例对象。

需要注意的是，


```java
/* ---------------------------------- FrameworkServlet ----------------------------------
 * 描述: 该类是HttpServletBean的子类,由springweb提供
 */
protected WebApplicationContext initWebApplicationContext() {  
	// 从ServletContext中获取
    WebApplicationContext rootContext =  
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());  
    //     
    WebApplicationContext wac = null;  

	// 第一个来源: 在通过构造函数构造FrameworkServlet实例时指定
	// this.webApplicationContext在通过构造函数构造FrameworkServlet实例时指定
    if (this.webApplicationContext != null) {
	    wac = this.webApplicationContext;  
        if (wac instanceof ConfigurableWebApplicationContext) {  
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;  
		    if (!cwac.isActive()) {
       
	            if (cwac.getParent() == null) {  
           
	                cwac.setParent(rootContext);  
                }  
                configureAndRefreshWebApplicationContext(cwac);  
            }  
        }  
    }  
    // 第二个来源: 从ServletContext中获取this.contextAttribute对应的WebApplicationContext对象
    if (wac == null) {  
        wac = findWebApplicationContext();  
    }  
    // 第三个来源: 创建this.contextClass(Class<?>)的实例 
    if (wac == null) {  
        // No context instance is defined for this servlet -> create a local one  
        wac = createWebApplicationContext(rootContext);  
    }  

    if (!this.refreshEventReceived) {  
        synchronized (this.onRefreshMonitor) {  
            onRefresh(wac);  
        }
    }  

    // Publish the context as a servlet context attribute.  
    if (this.publishContext) {  
		// getServletContextAttributeName(): 
        String attrName = getServletContextAttributeName();  
        getServletContext().setAttribute(attrName, wac);  
    }  
  
    return wac;  
}
```

下面给出WebApplicationContext每个来源的具体代码。

1）创建FrameworkServlet实例时，通过构造函数指定。
```java
/* ---------------------------------- FrameworkServlet ----------------------------------
 * 描述: 该类是HttpServletBean的子类,由springweb提供
 */
public FrameworkServlet(WebApplicationContext webApplicationContext) {  
    this.webApplicationContext = webApplicationContext;  
}
```

2）从ServletContext中获取 `this.contextAttribute(String)` 对应的WebApplicationContext对象。
```java
/* ---------------------------------- FrameworkServlet ----------------------------------
 * 描述: 该类是HttpServletBean的子类,由springweb提供
 */
protected WebApplicationContext findWebApplicationContext() {  
	// 获取 this.contextAttribute
    String attrName = getContextAttribute();  
    if (attrName == null) {  
        return null;  
    }  
    WebApplicationContext wac =  
         WebApplicationContextUtils.getWebApplicationContext(getServletContext(), attrName);  
    if (wac == null) {
      throw new IllegalStateException("No WebApplicationContext found: initializer not registered?");  
    }  
    return wac;  
}

/* ---------------------------------- WebApplicationContextUtils ----------------------------------
 */
public static WebApplicationContext getWebApplicationContext(ServletContext sc, String attrName) {  
	... // assert
    Object attr = sc.getAttribute(attrName);  
    if (attr == null) {  
        return null;  
    }  
    if (attr instanceof RuntimeException) {  
        throw (RuntimeException) attr;  
    }  
    if (attr instanceof Error) {  
        throw (Error) attr;  
    }  
    if (attr instanceof Exception) {  
        throw new IllegalStateException((Exception) attr);  
    }  
    if (!(attr instanceof WebApplicationContext)) {  
        throw new IllegalStateException("Context attribute is not of type WebApplicationContext: " + attr);  
    }  
    return (WebApplicationContext) attr;  
}
```


3）直接创建 `this.contextClass(Class<?>)` 指定的类的实例。
```java
/* ---------------------------------- FrameworkServlet ----------------------------------
 * 描述: 该类是HttpServletBean的子类,由springweb提供
 */
protected WebApplicationContext createWebApplicationContext(@Nullable WebApplicationContext parent) {  
    return createWebApplicationContext((ApplicationContext) parent);  
}
protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
	// 1. 获取this.contextClass
	// 2. 默认为 XmlWebApplicationContext.class
    Class<?> contextClass = getContextClass();  
    if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {  
		... // throws..
    }  
    ConfigurableWebApplicationContext wac =  
         (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);  
  
    wac.setEnvironment(getEnvironment());  
    wac.setParent(parent);  
    String configLocation = getContextConfigLocation();  
    if (configLocation != null) {  
        wac.setConfigLocation(configLocation);  
    }  
    configureAndRefreshWebApplicationContext(wac);  
  
    return wac;  
}
```


# 配置、refresh--configureAndRefreshWebApplicationContext()

```java
/* ---------------------------------- FrameworkServlet ----------------------------------
 * 描述: 该类是HttpServletBean的子类,由springweb提供
 */
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {  
    if (ObjectUtils.identityToString(wac).equals(wac.getId())) {  
        // The application context id is still set to its original default value  
        // -> assign a more useful id based on available information      
		if (this.contextId != null) {  
            wac.setId(this.contextId);  
        }  
        else {  
            // Generate default id...  
            wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +  
               ObjectUtils.getDisplayString(getServletContext().getContextPath()) + '/' + getServletName());  
        }  
    }  


    wac.setServletContext(getServletContext());  
    wac.setServletConfig(getServletConfig());  
    wac.setNamespace(getNamespace());
    wac.addApplicationListener(new SourceFilteringListener(wac, new ContextRefreshListener()));  
  
    // The wac environment's #initPropertySources will be called in any case when the context  
    // is refreshed; do it eagerly here to ensure servlet property sources are in place for   
    // use in any post-processing or initialization that occurs below prior to #refresh   
    ConfigurableEnvironment env = wac.getEnvironment();  
    if (env instanceof ConfigurableWebEnvironment) {
      ((ConfigurableWebEnvironment) env).initPropertySources(getServletContext(), getServletConfig());  
    }  

	// 1. 在
    // 2. 默认为NoOp方法
    postProcessWebApplicationContext(wac);  
    // 
    applyInitializers(wac);  
    
    wac.refresh();  
}
```


```java
protected void applyInitializers(ConfigurableApplicationContext wac) {  
	// 1. GLOBAL_INITIALIZER_CLASSES_PARAM = globalInitializerClasses
    String globalClassNames = getServletContext().getInitParameter(ContextLoader.GLOBAL_INITIALIZER_CLASSES_PARAM);  
    if (globalClassNames != null) {  
	    // 1. INIT_PARAM_DELIMITERS = ",; \t\n"
        for (String className : StringUtils.tokenizeToStringArray(globalClassNames, INIT_PARAM_DELIMITERS)) { 
	        // 1. 
            this.contextInitializers.add(loadInitializer(className, wac));  
        }  
    }  
  
    if (this.contextInitializerClasses != null) {  
	    // 1. INIT_PARAM_DELIMITERS = ",; \t\n"
		for (String className : StringUtils.tokenizeToStringArray(this.contextInitializerClasses, INIT_PARAM_DELIMITERS)) {  
            this.contextInitializers.add(loadInitializer(className, wac));  
        }  
    }

	// 
    AnnotationAwareOrderComparator.sort(this.contextInitializers);  
    for (ApplicationContextInitializer<ConfigurableApplicationContext> initializer : this.contextInitializers) {  
        initializer.initialize(wac);  
    }  
}
```

