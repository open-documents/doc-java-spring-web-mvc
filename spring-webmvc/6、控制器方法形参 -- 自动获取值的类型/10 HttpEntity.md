
HttpEntity表示http的request和resposne实体，它由消息头和消息体组成。
从HttpEntity中可以获取http请求头和回应头，也可以获取http请求体和回应体信息。
HttpEntity的使用，与@RequestBody 、@ResponseBody类似。

```java
@PostMapping("/accounts") 
public void handle(HttpEntity<Account> entity) {}
```


# 例子

在login.jsp发送ajax请求，发送之前添加请求头信息。
```javascript
$.ajax({
    type: "POST",
    url: targetUrl,
    data: user,//传递的参数
    dataType:"json",//前端可以接收服务器传过来的数据的类型，json
    contentType: "application/json",
    beforeSend: function(xhr){//请求发送之前执行函数,添加请求头
        xhr.setRequestHeader("token","shfashfdasfhdashfoasf");
    },
    success: function(user){
        alert(user.username);
    },
    error:function(){
        alert("异常，请检查");
    }
```
action中提取RequestEntity中的请求头信息，并用ResponseEntity回应
```java
@PostMapping(value = "/login")
@ResponseBody//返回的结果是响应体，返回的类型String,字符串中有中文，乱码了
public ResponseEntity<User> login(@RequestBody String userString, RequestEntity requestEntity) {
    System.out.println(requestEntity.getUrl());
    //通过请求实体对象获取请求头
    HttpHeaders requestHeaders = requestEntity.getHeaders();
    System.out.println(requestHeaders);
    System.out.println(requestHeaders.getContentLength());//内容的长度
    System.out.println(requestHeaders.getContentType());
    System.out.println(requestHeaders.getAccept());
    System.out.println(requestHeaders.getOrigin());
    System.out.println(requestHeaders.getFirst("token"));
 
    //创建一个响应头
    HttpHeaders responseHeader = new HttpHeaders();
    responseHeader.set("myResponseHeader","myValue");
    //userString绑定的请求体
    //登录成功，拿到一个user对象
    User user = new User();
    user.setUsername(userString.split("&")[0].split("=")[1]);
    user.setPassword(userString.split("&")[1].split("=")[1]);
    user.setId(1000001);
    user.setHead_img("/images/head.jpg");
    user.setBalance(1000);
    //现在返回登录的结果，json格式的字符串
    //user：响应体，responseHeader：响应头，本身就有默认的响应头，在这里是添加其他的响应头的信息
    //HttpStatus:响应的状态码
    return new ResponseEntity<User>(user, responseHeader, HttpStatus.OK);
}
```








