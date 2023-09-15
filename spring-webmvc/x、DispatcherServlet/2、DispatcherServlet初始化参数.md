	参考文档:P870

能够自定义DispatcherServlet实例，通过修改web.xml中的servlet初始化参数。

DispatcherServlet有如下初始化参数能够修改：

# 一、contextClass

该属性来自DispatcherServlet父类FrameworkServlet。

实现ConfigurableWebApplicationContext接口的类，当前的servet用它来创建上下文。如果这个参数没有指定，默认使用XmlWebApplicationContext。

# 二、WebApplicationContext

## contextConfigLocation

传给上下文实例(由contextClass指定)的字符串，用来指定上下文的位置。
- 当contextClass是XmlWebApplicationContext时，传入Spring配置文件位置。
- 当contextClass是AnnotationConfigWebApplicationContext时，传入Spring配置类位置。

这个字符串可以被分成多个字符串(使用逗号作为分隔符)来支持多个上下文(在多个上下文的情况下，如果同一个bean被定义两次，后面一个优先)。


```xml
<servlet>
	<servlet-name></servlet-name>
	<servlet-class></servlet-class>
	<load-on-startup></load-on-startup>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classPath:spring-servlet.xml</param-value>
	</init-param>
</servlet>
```

3.namespace:WebApplicationContext命名空间。默认值是`[server-name]-servlet`。

throwExceptionIfNoHandlerFound：当没有handler匹配当前请求时是否抛出NoHandlerFoundException。默认为false，此时DispatcherServlet将状态码设置为404。这个请求能被HandlerExceptionResolver捕获

# 映射路径

`/` ：所有请求。不包括.jsp后缀的请求。
`/*`：所有请求。包括.jsp后缀的请求。