
如果在@Controller类或其合成注解中声明@ExceptionHandler、@InitBinder、@ModelAttribute方法，它们只适用于这个@Controller类。

如果在@ControllerAdvice类或@RestControllerAdvice中声明@ExceptionHandler、@InitBinder、@ModelAttribute方法，它们适用于所有@Controller类。

从5.3开始，在@ControllerAdvice类中@ExceptionHandler方法能够处理来自其他控制器或handler的异常。

@ControllerAdvice是被@Component注解标注的元注解。
@RestControllerAdvice是被@ControllerAdvice和@ResponseBody注解标注的元注解。


RequestMappingHandlerMapping和ExceptionHandlerExceptionResolver一起在启动时检测@ControllerAdvice bean，并把它们用于运行时。

@ControllerAdvice中的@ExceptionHandler方法运行在局部@ExceptionHandler方法（即@Controller类中的@ExceptionHandler方法）之后。
@ControllerAdvice中的@InitBinder、@ModelAttribute方法方法运行在局部@InitBinder、@ModelAttribute方法（即@Controller类中的@InitBinder、@ModelAttribute方法）之前。


# 控制应用范围

@ControllerAdvice注解有属性可以控制它们适用到的controller和handler的范围。

在运行时会运行选择器，这样可能会影响性能。详细见[@ControllerAdvice](https://docs.spring.io/spring-framework/docs/6.0.3/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html)。

适用于所有被@RestController注解标注的Controller。
```java
// Target all Controllers annotated with @RestController 
@ControllerAdvice(annotations = RestController.class) 
public class ExampleAdvice1 {}
```
适用于`org.example.controllers`包下的controller。
```java
// Target all Controllers within specific packages 
@ControllerAdvice("org.example.controllers") public class ExampleAdvice2 {} 
```
适用于来自特定类子类的controller。
```java
// Target all Controllers assignable to specific classes 
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class}) 
public class ExampleAdvice3 {}
```

