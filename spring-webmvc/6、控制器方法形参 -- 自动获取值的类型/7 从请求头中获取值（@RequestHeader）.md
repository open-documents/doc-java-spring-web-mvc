

使用@RequestHeader注解能够绑定请求头参数到控制器方法参数。
- 如果控制器方法参数不是String，会自动进行类型转换。
- 如果控制器方法参数类型是Map<String, String>或MultiValueMap<String, String>或HttpHeaders，控制器方法参数会用所有请求头参数来填充。
- 如果请求头参数值是一个用逗号分割的字符串时，控制器方法参数可以是String、`String[]`、`List<String>`，后两者会被请求头参数值每隔一个逗号进行填充，如果是其他类型数组或list，还会进行类型转换。

# 例子

如果有下面一个请求报文：
```text
GET /learning_springmvc/test2 HTTP/1.1
User-Agent: PostmanRuntime/7.31.3
Accept: */*
Postman-Token: 45c8fabb-7f39-465e-8a17-bfdf82d65c4b
Host: localhost:8080
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```
下面一个例子获取请求头中的Host参数值：
```java
@RequestMapping("/test2")  
public String test2(@RequestHeader("Host")String host){  
    System.out.println(host);  
    return "success";  
}
// localhost:8080
```





