
# 

```java
@AutoConfiguration  
@EnableConfigurationProperties(ServerProperties.class)  
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)  
@ConditionalOnClass(CharacterEncodingFilter.class)  
@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)  
public class HttpEncodingAutoConfiguration {
	@Bean  
	@ConditionalOnMissingBean  
	public CharacterEncodingFilter characterEncodingFilter() {  
	   CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();  
	   filter.setEncoding(this.properties.getCharset().name());  
	   filter.setForceRequestEncoding(this.properties.shouldForce(Encoding.Type.REQUEST));  
	   filter.setForceResponseEncoding(this.properties.shouldForce(Encoding.Type.RESPONSE));  
	   return filter;  
	}
}
```


# 

```java
@AutoConfiguration(after = { DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,  
      ValidationAutoConfiguration.class })  
@ConditionalOnWebApplication(type = Type.SERVLET)  
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })  
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)  
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)  
@ImportRuntimeHints(WebResourcesRuntimeHints.class)  
public class WebMvcAutoConfiguration {
	@Bean  
	@ConditionalOnMissingBean(FormContentFilter.class)  
	@ConditionalOnProperty(prefix = "spring.mvc.formcontent.filter", name = "enabled", matchIfMissing = true)  
	public OrderedFormContentFilter formContentFilter() {  
	    return new OrderedFormContentFilter();  
	}
}


```

```java
@Configuration(proxyBeanMethods = false)  
@Import(EnableWebMvcConfiguration.class)  
@EnableConfigurationProperties({ WebMvcProperties.class, WebProperties.class })  
@Order(0)  
public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer, ServletContextAware {
	@Bean  
	@ConditionalOnMissingBean({ RequestContextListener.class, RequestContextFilter.class })  
	@ConditionalOnMissingFilterBean(RequestContextFilter.class)  
	public static RequestContextFilter requestContextFilter() {  
	    return new OrderedRequestContextFilter();  
	}
}
```