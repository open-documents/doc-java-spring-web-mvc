	参考文档:P1034

有下面两种方法导入默认配置：
- 使用@EnableWebMvc注解。
- 继承WebMvcConfigurationSupport类，并按需重写方法，最后注册为bean。

下面来展开介绍。

# 1.1）@EnableWebMvc注解

在@Configuration类上添加@EnableWebMvc注解，表示从DelegatingWebMvcConfiguration类中导入springmvc配置。
```java
@Configuration 
@EnableWebMvc 
public class WebConfig { }
```

# 1.2）<mvc:annotation-driven/>标签

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:mvc="http://www.springframework.org/schema/mvc"   
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	   xsi:schemaLocation="http://www.springframework.org/schema/beans  
	   https://www.springframework.org/schema/beans/spring-beans.xsd
	   http://www.springframework.org/schema/mvc
	   https://www.springframework.org/schema/mvc/spring-mvc.xsd">
	<mvc:annotation-driven/>
</beans>
```



# 继承WebMvcConfigurationSupport类

这是另外一种更加高级的用法，通过继承WebMvcConfigurationSupport类，并按需重写其中方法，然后将其注册为bean。