
Cookie在每次请求中被包含在请求头中，一个带有Cookie的请求头如下：
```text
Cookie:JSESSIONID=BC22339890A2E00E3C8F4B1D29F64C6D;Idea-cde08f12=1cb8877e-c3ce-40af-abc0-ab1471b37b5e
```


Springmvc提供 <font color="00E0FF">@CookieValue注解</font> 从请求头的Cookie数组中获取指定的Cookie。如果控制器方法参数不是String类型会自动进行类型转换。

# 例子

请求头中有如下键值对：


通过下面方式获取Cookie值：
```java
@RequestMapping("/test")  
public String test(@CookieValue("JSESSIONID")String JSESSIONID){  
    System.out.println(JSESSIONID);  
    return "success";  
}
// BC22339890A2E00E3C8F4B1D29F64C6D
```

