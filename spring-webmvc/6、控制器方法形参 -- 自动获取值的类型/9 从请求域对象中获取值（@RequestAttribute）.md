
使用@RequestAttribute注解获取已经存在的请求属性
```java
@GetMapping("/") 
public String handle(@RequestAttribute Client client) {}
```

@RequestAttribute值并不是通过前端传递，而是来自下面（httpServletRequest.setAttribute(name, value)）：
- 在Filter中创建。
- 在HandlerInterceptor创建。
