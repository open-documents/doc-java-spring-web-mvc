RouterFunctions.Builder的默认实现类。

RouterFunctionBuilder维护三个列表：
- 一个<font color=44cf57>存储RouterFunction</font>实例的列表<font color=44cf57>routerFunctions</font>。
- 一个<font color=44cf57>存储HandlerFilterFunction</font>实例的列表<font color=44cf57>filterFunctions</font>。




# 方法

| 签名                                                     | 描述                                                             |
| -------------------------------------------------------- | ---------------------------------------------------------------- |
| `GET(HandlerFunction<ServerResponse>)`                   | 使用HandlerFunction处理所有GET请求。                             |
| `GET(String, HandlerFunction<ServerResponse>)`           | 使用HandlerFunction处理请求路径为String的GET请求。               |
| `GET(RequestPredicate, HandlerFunction<ServerResponse>)` | 使用HandlerFunction处理请求满足一定条件(例如accept头)的GET请求。 |
|`GET(String, RequestPredicate, HandlerFunction<ServerResponse>)` |使用HandlerFunction处理请求路径为String、请求满足一定条件的GET请求。|



- `POST(String, HandlerFunction<ServerResponse>)`：为一个POST请求创建映射。

# 路由顺序

	参考文档:P951

与通过注解进行匹配的方式不同，RouterFunction中的路由先匹配第一个条件，第一个不满足然后进行尝试匹配第二个，以此类推。因此需要将范围小的路由放在更加泛化的路由之前。

```java
import static org.springframework.http.MediaType.APPLICATION_JSON; 
import static org.springframework.web.servlet.function.RequestPredicates.*; 
import static org.springframework.web.servlet.function.RouterFunctions.route;

PersonRepository repository = ... PersonHandler 
handler = new PersonHandler(repository); 
RouterFunction<ServerResponse> route = route()
					.GET("/person/{id}", accept(APPLICATION_JSON), handler::getPerson)   
					.GET("/person", accept(APPLICATION_JSON), handler::listPeople)   
					.POST("/person", handler::createPerson)
					.build(); 
public class PersonHandler {   
	public ServerResponse listPeople(ServerRequest request) {}   
	public ServerResponse createPerson(ServerRequest request) {}   
	public ServerResponse getPerson(ServerRequest request) {}
}
```


# 嵌套的RouteFunction
```java
RouterFunction<ServerResponse> route = route()
	.path("/person", builder -> builder ①   
		.GET("/{id}", accept(APPLICATION_JSON), handler::getPerson)   
		.GET(accept(APPLICATION_JSON), handler::listPeople)   
		.POST(handler::createPerson))   
		.build();
```

```java
RouterFunction<ServerResponse> route = route()   
	.path("/person", b1 -> b1   
		.nest(accept(APPLICATION_JSON), b2 -> b2   
			.GET("/{id}", handler::getPerson)   
			.GET(handler::listPeople))   
		.POST(handler::createPerson))   
	.build();
```


# 与HandlerFunctionFilter有关的方法

有3个与HandlerFilterFunction有关的方法：

<font color=fb463c>before()和after()都只对当前层面和该层以下层面的请求起作用。</font>

## filter()

定义：
```java
public RouterFunctions.Builder filter(HandlerFilterFunction<ServerResponse, ServerResponse> filterFunction) {  
	this.filterFunctions.add(filterFunction);  
	return this;
}
```
参数：
- 一个`HandlerFilterFunction`类型参数。

可以看到是向<font color=44cf57>filterFunctions数组</font>中添加一个<font color=44cf57>HandlerFilterFunction对象</font>。

-- --
## before()

定义：
```java
public RouterFunctions.Builder before(Function<ServerRequest, ServerRequest> requestProcessor) {  
	return filter(HandlerFilterFunction.ofRequestProcessor(requestProcessor));  
}
```
参数：
- 一个`Function<ServerRequest, ServerRequest>类型`参数。处理请求，返回处理后的请求。

可以看到本质还是向<font color=44cf57>filterFunctions数组</font>中添加一个<font color=44cf57>HandlerFilterFunction对象</font>。只不过这个<font color=44cf57>HandlerFilterFunction对象</font>不是人为定义的，而是HandlerFilterFunction接口预先定义好的一种特殊HandlerFilterFunction对象。

例子：
```java
RouterFunction<ServerResponse> route = route()
	.path("/person", 
		b1 -> b1.nest(accept(APPLICATION_JSON), b2 -> b2   
					.GET("/{id}", handler::getPerson)   
					.GET(handler::listPeople)   
					.before(request -> ServerRequest.from(request).header("X-RequestHeader", "Value")
					                                              .build()))
				.POST(handler::createPerson))
	.build();
```

-- --
## after()

定义：
```java
public RouterFunctions.Builder after(BiFunction<ServerRequest, ServerResponse, ServerResponse> responseProcessor) {  
	return filter(HandlerFilterFunction.ofResponseProcessor(responseProcessor));  
}
```


可以看到本质还是向<font color=44cf57>filterFunctions数组</font>中添加一个<font color=44cf57>HandlerFilterFunction对象</font>。只不过这个<font color=44cf57>HandlerFilterFunction对象</font>不是人为定义的，而是HandlerFilterFunction接口预先定义好的一种特殊HandlerFilterFunction对象。

例子：
```java
RouterFunction<ServerResponse> route = route()
	.path("/person")
	.after((request, response) -> logResponse(response))
	.build()
```

