
由于SpringServletContainerInitializer本质上调用了WebApplicationInitializer接口实现类来初始化Web应用环境，所以下面来看spring-web模组中提供的3个重要的WebApplicationInitializer接口实现类。

# 1 AbstractContextLoaderInitializer

先来看定义，可以看到AbstractContextLoaderInitializer是WebApplicationInitializer接口的直接实现抽象类。
```java
public abstract class AbstractContextLoaderInitializer implements WebApplicationInitializer {}
```
再来看重要的onStartup()方法。主要完成了2件事：
- 创建springApplicationContext。因为spring-web是基于spring的，因此创建一个WebApplicationContext是至关重要的。（这件事情也是该抽象类名称的来源）
- 向容器中

```java
/* ----------------------------- AbstractContextLoaderInitializer -----------------------------
*/
public void onStartup(ServletContext servletContext) throws ServletException {  
	registerContextLoaderListener(servletContext);  
}
/* ----------------------------- AbstractContextLoaderInitializer -----------------------------
*/
protected void registerContextLoaderListener(ServletContext servletContext) {  
	// 抽象方法,需要子类重写
	// 在子子类AbstractAnnotationConfigDispatcherServletInitializer被重写
	WebApplicationContext rootAppContext = createRootApplicationContext();  
    if (rootAppContext != null) {  
	    // 在Servlet容器启动阶段,ServletContext初始化后,接收到ServletContext初始化事件,用来初始化Root 
	    // WebApplicationContext
        ContextLoaderListener listener = new ContextLoaderListener(rootAppContext);  
        listener.setContextInitializers(getRootApplicationContextInitializers());  
        servletContext.addListener(listener);  
    }  
    else {  
	    ... // logger.debug("No ContextLoaderListener registered, as createRootApplicationContext() did not
		    // return an application context");  
    }  
}
```

# 2 AbstractDispatcherServletInitializer

先来看定义，可以看到继承了前面提到的AbstractContextLoaderInitializer。
```java
public abstract class AbstractDispatcherServletInitializer extends AbstractContextLoaderInitializer{}
```
再来看重要的onStartup()方法。
```java
/* ---------------------------- AbstractDispatcherServletInitializer ----------------------------
 * 
 */
public void onStartup(ServletContext servletContext) throws ServletException { 
	// 调用AbstractContextLoaderInitializer#onStartup()
    super.onStartup(servletContext);  
    registerDispatcherServlet(servletContext);  
}
/* ---------------------------- AbstractDispatcherServletInitializer ----------------------------
 * 
 */
protected void registerDispatcherServlet(ServletContext servletContext) {  
	// protected方法,默认返回"dispatcher"
    String servletName = getServletName();  
    Assert.hasLength(servletName, "getServletName() must not return null or empty");  

	// 创建
    WebApplicationContext servletAppContext = createServletApplicationContext();  
    ... // assert: "createServletApplicationContext() must not return null");  

    FrameworkServlet dispatcherServlet = createDispatcherServlet(servletAppContext);  
    ... // assert: "createDispatcherServlet(WebApplicationContext) must not return null"); 
	// getServletApplicationContextInitializers(): 默认返回null
    dispatcherServlet.setContextInitializers(getServletApplicationContextInitializers());  

	// 向ServletContext中添加Servlet
    ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);  
    if (registration == null) {  
	    throw new IllegalStateException("Failed to register servlet with name '" + servletName + "'. " +  
	        "Check if there is another servlet registered under the same name.");  
    }  
    // 设置DispatcherServlet启动级别
    registration.setLoadOnStartup(1);  
    // 添加DispatcherServlet路径映射
    registration.addMapping(getServletMappings());  
    // 
    registration.setAsyncSupported(isAsyncSupported());  

	// getServletFilters(): protected方法,默认返回null
    Filter[] filters = getServletFilters();  
    if (!ObjectUtils.isEmpty(filters)) {  
	    for (Filter filter : filters) {  
		    // 向ServletContext中注册过滤器
            registerServletFilter(servletContext, filter);  
        }  
    }  

	// 进一步自定义DispatcherServlet在ServletContext中的属性
    customizeRegistration(registration);  
}

```
## 相关方法--registerServletFilter()
```java
protected FilterRegistration.Dynamic registerServletFilter(ServletContext servletContext, Filter filter) {  
    String filterName = Conventions.getVariableName(filter);  
    Dynamic registration = servletContext.addFilter(filterName, filter);  
  
    if (registration == null) {  
        int counter = 0;  
        while (registration == null) {  
            if (counter == 100) {  
	            throw new IllegalStateException("Failed to register filter with name '" + filterName + "'. " 
	            + "Check if there is another filter registered under the same name.");  
            }  
            registration = servletContext.addFilter(filterName + "#" + counter, filter);  
            counter++;  
        }  
    }  
  
    registration.setAsyncSupported(isAsyncSupported());  
    registration.addMappingForServletNames(getDispatcherTypes(), false, getServletName());  
    return registration;  
}
```

# 3 AbstractAnnotationConfigDispatcherServletInitializer

先来看定义。
```java
public abstract class AbstractAnnotationConfigDispatcherServletInitializer extends AbstractDispatcherServletInitializer
```
需要注意的是：该类中并没有对onStartup()进行功能增强，只是对。
```java
/* ---------------------- AbstractAnnotationConfigDispatcherServletInitializer ----------------------
 */
protected WebApplicationContext createRootApplicationContext() {  
   Class<?>[] configClasses = getRootConfigClasses();  
   if (!ObjectUtils.isEmpty(configClasses)) {  
      AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
      context.register(configClasses);  
      return context;  
   }  
   else {  
      return null;  
   }  
}

protected WebApplicationContext createServletApplicationContext() {  
   AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
   Class<?>[] configClasses = getServletConfigClasses();  
   if (!ObjectUtils.isEmpty(configClasses)) {  
      context.register(configClasses);  
   }  
   return context;  
}
```