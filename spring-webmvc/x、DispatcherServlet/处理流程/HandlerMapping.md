SpringMVC 根据 `HandlerMapping` 将 `Request` 和 `Controller` 里面的方法对应起来的。

`HandlerMapping` 根据请求匹配到对应的 `Handler`，然后将找到的 `Handler` 和所有匹配的 `HandlerInterceptor`（拦截器）绑定到创建的 `HandlerExecutionChain` 对象上并返回。  
`HandlerMapping` 只是一个接口类，不同的实现类有不同的匹对方式，根据功能的不同我们需要在 SpringMVC 容器中注入不同的映射处理器 `HandlerMapping`。

