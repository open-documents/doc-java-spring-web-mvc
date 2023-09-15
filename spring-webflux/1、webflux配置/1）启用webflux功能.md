有如下几种启用webflux功能的方法：
- @Configuration类上标注@EnableWebFlux注解。


# @EnableWebFlux注解

## 使用@EnableWebFlux注解启用webflux功能

通过在@Configuration类上标注@EnableWebFlux注解来启用webflux功能。
```java
@Configuration
@EnableWebFlux 
public class WebConfig { }
```
## 详解

下面是@EnableWebFlux注解的定义：
```java
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.TYPE)  
@Documented  
@Import(DelegatingWebFluxConfiguration.class)  
public @interface EnableWebFlux {  
}
```
可以看到，@EnableWebFlux注解的作用是向容器中注册DelegatingWebFluxConfiguration组件。


