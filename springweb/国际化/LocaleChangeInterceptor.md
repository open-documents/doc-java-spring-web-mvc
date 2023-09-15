位于`org.springframework.web.servlet.i18n包`。

LocaleChangeInterceptor允许根据请求参数（默认请求参数名称为locale）改变当前区域。

# 定义

```java
public class LocaleChangeInterceptor implements HandlerInterceptor {
	public static final String DEFAULT_PARAM_NAME = "locale";
	private String paramName = DEFAULT_PARAM_NAME;
}
```

# 基本使用

LocaleChangeInterceptor拦截器可以用来根据请求中的参数调用LocaleResolver中的setLocale()方法改变用户地区。

```xml
<bean id="localeChangeInterceptor" class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">   
	<property name="paramName" value="siteLanguage"/>
</bean>
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver"/>
<bean id="urlMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	<property name="interceptors"> 
		<list>
			<ref bean="localeChangeInterceptor"/>
		</list> 
	</property>  
	<property name="mappings">
		<value>/**/*.view=someController</value> 
	</property>
</bean>
```