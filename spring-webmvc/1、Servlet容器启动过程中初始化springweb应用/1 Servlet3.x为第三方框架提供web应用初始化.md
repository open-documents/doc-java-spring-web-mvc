
# 一、Servlet3.x提供的SPI服务

除了以往容器自身提供的容器初始化方式（即通过web.xml配置的方式），Servlet3.x 规范为第三方框架提供了初始化Servlet容器的SPI服务。

首先看一个与之相关的重要接口ServletContainerInitializer。
```java
/* ---------------------------- ServletContainerInitializer ----------------------------
 * Since: Servlet3.0
 */
public interface ServletContainerInitializer {
	public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException;
}
```

<font color=44cf57>在第三方框架定义该接口的实现类。当Servlet容器在自身启动过程中，会调用第三方框架提供的该接口实现类</font>，那么Servlet容器如何获知该接口的第三方框架实现类是哪个呢？

这就引出了Servlet3.x规范为第三方框架提供的初始化Servlet容器的SPI服务的<font color="FFDD00">第一条要求：第三方框架提供的ServletContainerInitializer接口实现类全限定名需要写入在 jar包下的</font> `META-INF/services/ServletContainerInitializer全限定名` <font color="FFDD00">文件中</font>（请注意：由于Servlet规范所处的项目在4.x以后发生了变化，所以需要分别对待）。

另外，Servlet容器会为第三方框架提供的实现类的onStartup()方法传递一组符合条件的类型为Class的参数，哪些类将会被传递进去呢？这就引出了Servlet3.x规范为第三方框架提供的初始化Servlet容器的SPI服务的<font color="FFDD00">第二条要求</font>：
<font color="FFDD00">第三方框架在提供的ServletContainerInitializer接口实现类上标注@HandlesTypes注解，在注解中指定一系列类，符合条件的类会被传递进onStartup()方法参数中。</font>

下面是符合的条件：
- 当@HandlesTypes注解中指定的Class是接口时，实现该接口的类都会被传递。
- 当@HandlesTypes注解中指定的Class是类时，继承该类的子类都会被传递。
- 当@HandlesTypes注解中指定的Class是注解时，被该注解标注的类都会被传递。

# 二、Springweb中的使用

以spring-web5.3.18，Servlet规范以3.1.0为例。

首先看Springweb模块如何满足第一条要求和第二条要求。下面是Spring-web模块提供的ServletContainerInitializer接口的实现类SpringServletContainerInitializer的定义。
```java
@HandlesTypes(WebApplicationInitializer.class)  
public class SpringServletContainerInitializer implements ServletContainerInitializer {
	@Override
	public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)  
throws ServletException {
		...
	}
}
```

先看spring-web模块如何满足第一条要求：
在spring-web模块下的 `META-INF\services\javax.servlet.ServletContainerInitializer` 文件中写入了下面内容。
```text
org.springframework.web.SpringServletContainerInitializer
```

再看spring-web如何满足第二条要求：
可以看到SpringServletContainerInitializer标注的@HandlesTypes注解中指定了WebApplicationInitializer.class（接口），因此Servlet容器在启动过程中，调用SpringServletContainerInitializer#onStartup()方法时会传递进WebApplicationInitializer接口的实现类给第一个参数。


