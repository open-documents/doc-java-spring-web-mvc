	参考文档:P984

默认情况下


# 使用Java配置

```java
@Configuration 
@EnableWebMvc 
public class WebConfig implements WebMvcConfigurer {   
	@Override   
	public void addCorsMappings(CorsRegistry registry) {   
		registry.addMapping("/api/**")   
				.allowedOrigins("https://domain2.com")   
				.allowedMethods("PUT", "DELETE")   
				.allowedHeaders("header1", "header2", "header3")   
				.exposedHeaders("header1", "header2")   
				.allowCredentials(true)
				.maxAge(3600);   
				// Add more mappings...
	}
}
```


# 使用xml配置

```xml
<mvc:cors>   
	<mvc:mapping path="/api/**"  
				 allowed-origins="https://domain1.com, https://domain2.com" 
				 allowed-methods="GET, PUT" 
				 allowed-headers="header1, header2, header3"
				 exposed-headers="header1, header2"
				 allow-credentials="true" 
				 max-age="123" /> 
	<mvc:mapping path="/resources/**" allowed-origins="https://domain1.com" />
</mvc:cors>
```

# 使用Filter配置

```java
CorsConfiguration config = new CorsConfiguration(); 
// Possibly... 
// config.applyPermitDefaultValues() 
config.setAllowCredentials(true); 
config.addAllowedOrigin("https://domain1.com"); 
config.addAllowedHeader("*"); 
config.addAllowedMethod("*"); 
UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource(); 
source.registerCorsConfiguration("/**", config); 
CorsFilter filter = new CorsFilter(source);
```