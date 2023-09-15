	参考文档:P868~P869

AbstractDispatcherServletInitializer提供方法去添加Filter实例。
```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {
	// ...
	@Override 
	protected Filter[] getServletFilters() {
		return new Filter[] { new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
	}
}
```

- 每个Filter的名称基于Filter具体的类型名称。
- Filter自动匹配到DispathcerServlet。