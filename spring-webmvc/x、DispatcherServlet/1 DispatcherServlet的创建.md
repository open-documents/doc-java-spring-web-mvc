
从前面可知，DispatcherServlet的实例化是在WebApplicationInitializer接口的实现类AbstractDispatcherServletInitializer中创建的。
```java

/* ---------------------------- AbstractDispatcherServletInitializer ----------------------------
 * 
 */
protected void registerDispatcherServlet(ServletContext servletContext) {
	...
	// 默认返回new DispatcherServlet(servletAppContext)创建的实例
	FrameworkServlet dispatcherServlet = createDispatcherServlet(servletAppContext);  
	...
	// 向ServletContext中注册DispatcherServlet
	ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet); 
	...
}
```

# 构造函数

下面就来看下DispatcherServlet的构造函数。
```java
/* --------------------------------- DispatcherServlet ---------------------------------
 * 
 */
public DispatcherServlet(WebApplicationContext webApplicationContext) {  
    super(webApplicationContext);  
    // FrameworkServlet中的boolean属性dispatchOptionsRequest决定DispatcherServlet是否将Http OPTIONS请求分发到
    // doService()方法
    setDispatchOptionsRequest(true);  
}
/* --------------------------------- FrameworkServlet ---------------------------------
 * 描述: 该类是DispatcherServlet的父类
 */
public FrameworkServlet(WebApplicationContext webApplicationContext) {  
	// WebApplicationContext类型
    this.webApplicationContext = webApplicationContext;  
}

```