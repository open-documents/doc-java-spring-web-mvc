	参考文档:P1040~

# 注册拦截器

## java代码注册
```java
@Configuration 
@EnableWebMvc 
public class WebConfig implements WebMvcConfigurer {   
	@Override   
	public void addInterceptors(InterceptorRegistry registry) {   
		registry.addInterceptor(new LocaleChangeInterceptor());   
		registry.addInterceptor(new ThemeChangeInterceptor())
		.addPathPatterns("/**")
		.excludePathPatterns("/admin/**");
	}
}
```
## xml注册

```xml
<mvc:interceptors>   
	<bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
	<mvc:interceptor>
		<mvc:mapping path="/**"/>
		<mvc:exclude-mapping path="/admin/**"/>
		<bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor"/>
	</mvc:interceptor>
</mvc:interceptors>
```