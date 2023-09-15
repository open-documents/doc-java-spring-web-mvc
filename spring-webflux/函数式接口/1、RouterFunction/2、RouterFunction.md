	参考文档:P940、950

- RouterFunction等价于@RequestMapping注解，主要不同是，它不仅提供数据还提供行为。
- 当注册RouterFunction为bean时，它会被servlet自动检测。

# 定义

RouterFunction是一个函数式接口。
route()是其需要重写的方法。接受ServerRequest参数，返回HandlerFunction。
```java
@FunctionalInterface  
public interface RouterFunction<T extends ServerResponse> {
	Optional<HandlerFunction<T>> route(ServerRequest request);	
}
```


# 创建

使用RouterFunctions去得到一个RouterFunction。
- RouterFunctions.route()：返回一个RouterFunctionBuilder。
- RouterFunctions.route(RequestPredicate, HandlerFunction)：返回DefaultRouterFunction（RouterFunction的实现类，RouterFunctions中的静态类。



