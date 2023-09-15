- WebApplicationContext是ApplicationContext的扩展。
- WebApplicationContext有到ServletContext和与之相关的Servlet的连接。
- WebApplicationContext绑定到ServletContext，因此应该能通过RequestContextUtils中的静态方法找到WebApplicationContext。

# 一、Context层次

	参考文档:P861~

通常一个WebApplicationContext就足够了，但是也支持Context之间的层次结构。

Root WebApplicationContext被多个DispatcherServlet或其他Servlet所共享

在root WebApplicationContext中，通常定义基础功能bean，如持久层data repositories、business services这些在多个servlet内使用到的bean。这些bean能够被子WebApplicationContext继承和重写。

-- --
下面一个例子配置WebApplicationContext层次结构：
```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {   
	@Override 
	protected Class<?>[] getRootConfigClasses() {
		return new Class<?>[] { RootConfig.class };
	} 
	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class<?>[] { App1Config.class }; 
	} 
	@Override
	protected String[] getServletMappings() {  
		return new String[] { "/app1/*" }; 
	}
}
```
如果一个应用不需要层次结构，那么：
- getRootConfigClasses()：返回所有配置类。
- getServletConfigClasses()：返回null。
-- --
等价的web.xml：
```xml
<web-app>
	<listener>
		<listenerclass>org.springframework.web.context.ContextLoaderListener</listener-class> 
	</listener>
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/root-context.xml</param-value>
	</context-param>
	<servlet> 
		<servlet-name>app1</servlet-name> 
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servletclass>
		<init-param> 
			<param-name>contextConfigLocation</param-name> 
			<param-value>/WEB-INF/app1-context.xml</param-value> 
		</init-param>
		<load-on-startup>1</load-on-startup> 
	</servlet> 
	<servlet-mapping> 
		<servlet-name>app1</servlet-name> 
		<url-pattern>/app1/*</url-pattern>
	</servlet-mapping>
</web-app>
```
如果一个应用不需要层次结构，那么<init-param/>中的contextConfigLocation参数值为空即可。