	参考文档:P1223

通过配置类实现WebFluxConfigurer接口来自定义webflux配置。

```java
@Configuration 
@EnableWebFlux 
public class WebConfig implements WebFluxConfigurer {   
	// Implement configuration methods...
}
```

