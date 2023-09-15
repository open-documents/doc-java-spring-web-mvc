	参考文档:P923

@ResponseBody注解标注在方法上能够让参数的返回结果序列化到response body中。

```java
@GetMapping("/accounts/{id}") 
@ResponseBody 
public Account handle() {}
```

@ResponseBody注解能够标注在类上，类中方法会继承@ResponseBody注解。
@RestController是@Controller和@ResponseBody注解的合成注解。

其他特征：P977、P970