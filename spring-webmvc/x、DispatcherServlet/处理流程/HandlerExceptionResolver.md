	参考文档:P872~P873

当在处理请求映射过程中或者在处理请求过程中（request handler，例如一个@Controller），DispatcherServlet委托HandlerExceptionResolver beans链去处理这个异常。

HandlerExceptionResolver规定它的返回值是如下几种情况：
- 一个ModelAndView，指向一个错误视图。
- 一个空的ModelAndView，如果异常在HandlerExceptionResolver内部处理掉。
- null，如果一个HandlerExceptionResolver处理不了，HandlerExceptionResolver链的后一个HandlerExceptionResolver会尝试处理，如果都处理不了，允许向Servlet容器抛出。


# 、内置的HandlerExceptionResolver

| HandlerExceptionResolver          | 描述                                                                                 |
| --------------------------------- | ------------------------------------------------------------------------------------ |
| SimpleMappingExceptionResolver    | 处理一个异常类名称和错误试图之间的映射。通常用于错误页面的渲染。                     |
| DefaultHandlerExceptionResolver   | 处理Springmvc抛出的异常，并将它们映射为一个HTTP状态码。                              |
| ResponseStatusExceptionResolver   | 处理@ResponseStatus标注的方法抛出的异常，并将HTTP状态码设置成注解指定的值。          |
| ExceptionHandlerExceptionResolver | 处理异常，通过调用标注在@Controller或@ControllerAdvice内部的@@ExceptionHandler方法。 |

# 、HandlerExceptionResolver链


# 、定义错误页面

	参考文档:P873~P874

在web.xml中声明错误页面。
```xml
<error-page>
	<location>/error</location>
</error-page>
```

如果一个异常无法被整个HandlerExceptionResolver链处理或者response有一个错误状态码，Servlet容器将会导向一个配置好的URL（此处是/error），这个请求会被DispatcherServlet处理，可能会将这个请求映射到一个@Controller，这个@Controller可能返回一个错误视图名称或者返回一个JSON response。
```java
@RestController 
public class ErrorController {   
	@RequestMapping(path = "/error") 
	public Map<String, Object> handle(HttpServletRequest request) { 
		Map<String, Object> map = new HashMap<String, Object>(); 
		map.put("status", request.getAttribute("jakarta.servlet.error.status_code")); 
		map.put("reason", request.getAttribute("jakarta.servlet.error.message")); 
		return map;
	}
}
```
