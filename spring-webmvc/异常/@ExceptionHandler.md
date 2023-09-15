

@Controller和@ControllerAdvice类种能声明一个或多个@ExceptionHandler方法来处理控制器方法中出现的异常。

The exception may match against a top-level exception being propagated (e.g. a direct IOException being thrown) or against a nested cause within a wrapper exception (e.g. an IOException wrapped inside an IllegalStateException). As of 5.3, this can match at arbitrary cause levels, whereas previously only an immediate cause was considered.

For matching exception types, preferably declare the target exception as a method argument, as the preceding example shows. When multiple exception methods match, a root exception match is generally preferred to a cause exception match. More specifically, the ExceptionDepthComparator is used to sort exceptions based on their depth from the thrown exception type.

The distinction between root and cause exception matching can be surprising. 
In the IOException variant shown earlier, the method is typically called with the actual FileSystemException or RemoteException instance as the argument, since both of them extend from IOException. However, if any such matching exception is propagated within a wrapper exception which is itself an IOException, the passedin exception instance is that wrapper exception. 
The behavior is even simpler in the handle(Exception) variant. This is always invoked with the wrapper exception in a wrapping scenario, with the actually matching exception to be found through ex.getCause() in that case. The passed-in exception is the actual FileSystemException or RemoteException instance only when these are thrown as top-level exceptions. 
We generally recommend that you be as specific as possible in the argument signature, reducing the potential for mismatches between root and cause exception types. Consider breaking a multimatching method into individual @ExceptionHandler methods, each matching a single specific exception type through its signature.
In a multi-@ControllerAdvice arrangement, we recommend declaring your primary root exception mappings on a @ControllerAdvice prioritized with a corresponding order. While a root exception match is preferred to a cause, this is defined among the methods of a given controller or @ControllerAdvice class. This means a cause match on a higher-priority @ControllerAdvice bean is preferred to any match (for example, root) on a lower-priority @ControllerAdvice bean. 
Last but not least, an @ExceptionHandler method implementation can choose to back out of dealing with a given exception instance by rethrowing it in its original form. This is useful in scenarios where you are interested only in root-level matches or in matches within a specific context that cannot be statically determined. A rethrown exception is propagated through the remaining resolution chain, as though the given @ExceptionHandler method would not have matched in the first place. Support for @ExceptionHandler methods in Spring MVC is built on the DispatcherServlet level, HandlerExceptionResolver mechanism.



# 例子

```java
@Controller 
public class SimpleController {  
	@ExceptionHandler   
	public ResponseEntity<String> handle(IOException ex) {}
}
```

# 限制匹配的异常类型

可以直接在@ExceptionHandler注解中声明能够匹配的异常类型。
```java
@ExceptionHandler({FileSystemException.class, RemoteException.class}) 
public ResponseEntity<String> handle(IOException ex) {}
```

# 支持的方法参数类型

	参考文档:P936~P937

# 支持的返回类型

	参考文档:P937~938


