DelegatingWebMvcConfiguration重写了WebMvcConfigurationSupport中支持重写的所有方法用于添加自定义配置。

DelegatingWebMvcConfiguration中维护了一个WebMvcConfigurerComposite属性。
```java
@Configuration(proxyBeanMethods = false)  
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
	private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();
}	
```
而WebMvcConfigurerComposite中维护了一个WebMvcConfigurer的列表（所以叫做Composite）。
```java
class WebMvcConfigurerComposite implements WebMvcConfigurer {  
	private final List<WebMvcConfigurer> delegates = new ArrayList<>();
}	
```
DelegatingWebMvcConfiguration重写的所有方法将添加自定义配置的功能交由WebMvcConfigurerComposite属性完成，而WebMvcConfigurerComposite中完成添加自定义配置的功能是由维护的一系列WebMvcConfigurer完成，WebMvcConfigurer是真正完成添加自定义配置的类。

举个例子：以addArgumentResolvers()方法为栗。

下面是DelegatingWebMvcConfiguration中的addArgumentResolvers()方法。
```java
@Configuration(proxyBeanMethods = false)  
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

	private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

	@Override  
	protected void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {  
	    this.configurers.addArgumentResolvers(argumentResolvers);  // 交由WebMvcConfigurerComposite完成
	}
}
```

下面是WebMvcConfigurerComposite中的addArgumentResolvers()方法。
```java
class WebMvcConfigurerComposite implements WebMvcConfigurer {
	private final List<WebMvcConfigurer> delegates = new ArrayList<>();

	@Override  
	public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {  
	    for (WebMvcConfigurer delegate : this.delegates) {  
	        delegate.addArgumentResolvers(argumentResolvers);  // 交由一系列WebMvcConfigurer完成
        }  
	}
}
```