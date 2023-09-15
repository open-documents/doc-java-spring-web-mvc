
当通过请求体传参时（最常见的就是POST请求了），可以通过@RequestBody获取参数。

@RequestBody使用HttpMessageConverter将请求体中的参数反序列化为一个对象。

# 例子

```java
@PostMapping("/accounts") 
public void handle(@RequestBody Account account) {}
```

# 验证

@RequestPart注解可以与jakarta.validation.Valid或spring的@Validated注解一起使用，两者都能触发bean验证。
默认情况下，验证错误会导致MethodArgumentNotValidException，会被抓换成400（BAD_REQUEST）。
可以通过Errors或BindingResult参数在控制器方法中处理验证错误。
```java
@PostMapping("/accounts") 
public void handle(@Valid @RequestBody Account account, BindingResult result) {}
```