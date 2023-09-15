see also： **`org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer`**

下面是定义：
```java
public abstract class AbstractDispatcherServletInitializer extends AbstractContextLoaderInitializer {
	@Override  
	public void onStartup(ServletContext servletContext) throws ServletException {  
	    super.onStartup(servletContext);  
	    registerDispatcherServlet(servletContext);  
	}
}
```
可以看到，AbstractDispatcherServletInitializer除了在AbstractContextLoaderInitializer注册一个新的ContextLoader（也是监听器）：ContextLoaderListener的基础上，还<font color=44cf57>另外注册了一个新的DispatcherServlet（使用DispatcherServlet的构造函数，这个构造函数初始化了WebApplicationContext属性，该行为可以通过重写createDispatcherServlet()方法来改变），并对这个新的DispatcherServlet做了下面的工作：</font>
- 为该DispatcherServlet添加过滤器(Filter)。
- 将该DispatcherServlet注册到ServletContext的Servlet集合中。
- 设置了DispatcherServlet注册的属性（启动级别、异步、路径映射）。

让我们看一下重要的registerDispatcherServlet方法。

# 1）registerDispatcherServlet

下面是registerDispatcherServlet的定义：
```java
protected void registerDispatcherServlet(ServletContext servletContext) {  
    String servletName = getServletName();  
    // 必须重写的方法--createServletApplicationContext
    WebApplicationContext servletAppContext = createServletApplicationContext();
    // 可以重写但不推荐的方法--createDispatcherServlet
    // 有默认实现: return new DispatcherServlet(servletAppContext);  
    FrameworkServlet dispatcherServlet = createDispatcherServlet(servletAppContext);
    // 可以重写的方法--getServletApplicationContextInitializers
    dispatcherServlet.setContextInitializers(getServletApplicationContextInitializers());
    // servletName是该类的一个属性，定义了新建的DispatcherServlet的名称
    ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);  
    registration.setLoadOnStartup(1);  
    // 3 必须重写的方法--getServletMappings
    registration.addMapping(getServletMappings());  
    registration.setAsyncSupported(isAsyncSupported());  
    
    Filter[] filters = getServletFilters();  
    if (!ObjectUtils.isEmpty(filters)) {  
        for (Filter filter : filters) {  
            registerServletFilter(servletContext, filter);  
        }   
    }  
    
    customizeRegistration(registration);  
}
```

可以看到影响注册的新的DispatcherServlet的两个方法：
- createServletApplicationContext
- getServletApplicationContextInitializers
-- --
# 必须重写的方法

下面来看下必须重写的方法。
## 1.1）createServletApplicationContext()
```java
protected abstract WebApplicationContext createServletApplicationContext();
```
在registerDispatcherServlet()方法中被调用，返回值会被用在新建DispatcherServlet时的构造函数中。
ServletApplicationContext一般包含控制器bean、视图解析器bean、locale解析器bean等。
## 1.2）getServletMappings()
```java
protected abstract String[] getServletMappings();
```
在registerDispatcherServlet()方法中被调用，用来向新建的DispatcherServlet中添加mapping。

## 2）来自父类AbstractContextLoaderInitializer的方法

下面是来自父类AbstractContextLoaderInitializer的<font color=44cf57>必须重写</font>的方法：
- createRootApplicationContext()

-- -- 
接下来看下可以重写的方法。
# 可以重写的方法

## 1.1）getServletApplicationContextInitializers()

```java
@Nullable  
protected ApplicationContextInitializer<?>[] getServletApplicationContextInitializers() {  
    return null;  
}
```
## 1.2）getServletFilters()
```java
@Nullable  
protected Filter[] getServletFilters() {  
    return null;  
}
```


## 1.3）customizeRegistration()
```java
protected void customizeRegistration(ServletRegistration.Dynamic registration) {}
```
在registerDispatcherServlet()方法中，已经针对添加到ServletContext中的新建的DispatcherServlet做了如下处理：
- 设置<font color=44cf57>启动级别</font>为1。registration.setLoadOnStartup(1)。
- <font color=44cf57>添加映射路径</font>。registration.addMapping(getServletMappings())，可以看到要添加新的映射路径只需要重写getServletMappings()方法。
- 设置<font color=44cf57>是否支持异步</font>。setAsyncSupported(isAsyncSupported())

## 1.4）isAsyncSupported()
```java
protected boolean isAsyncSupported() { return true; }
```
在registerDispatcherServlet中被调用（ServletRegistration.Dynamic#setAsyncSupported(isAsyncSupported()))。设置新建的DispathcerServlet是否支持异步。

## 1.5）createDispatcherServlet()（不推荐）

```java
protected FrameworkServlet createDispatcherServlet(WebApplicationContext servletAppContext) {  
    return new DispatcherServlet(servletAppContext);  
}
```

## 2）来自父类AbstractContextLoaderInitializer的方法

下面是来自父类AbstractContextLoaderInitializer的<font color=44cf57>可以重写</font>的方法：
- getRootApplicationContextInitializers()



