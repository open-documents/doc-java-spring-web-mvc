ResponseEntity在HttpEntity的基础上添加了状态码。关于HttpEntity，见

下面是ResponseEntity的定义：
```java
public class ResponseEntity<T> extends HttpEntity<T> {
	private final Object status;
}
```
## 构造函数

下面是其构造函数：
```java
// 仅用状态码
public ResponseEntity(HttpStatus status) {  this(null, null, status);  }
// 仅用报文体和状态码
public ResponseEntity(@Nullable T body, HttpStatus status) {  this(body, null, status); }
// 仅用报文头和状态码
public ResponseEntity(MultiValueMap<String, String> headers, HttpStatus status) {  
   this(null, headers, status);  
}
public ResponseEntity(@Nullable T body, @Nullable MultiValueMap<String, String> headers, HttpStatus status) {  
   this(body, headers, (Object) status);  
}
public ResponseEntity(@Nullable T body, @Nullable MultiValueMap<String, String> headers, int rawStatus) {  
   this(body, headers, (Object) rawStatus);  
}
// 私有构造函数
private ResponseEntity(@Nullable T body, @Nullable MultiValueMap<String, String> headers, Object status) {  
   super(body, headers);  
   Assert.notNull(status, "HttpStatus must not be null");  
   this.status = status;  
}
```

## getter

```java
public HttpStatus getStatusCode()
public int getStatusCodeValue()
```

## 设置报文体

```java

public static BodyBuilder status(HttpStatus status)
public static BodyBuilder status(int status)
public static BodyBuilder ok()
public static BodyBuilder accepted()
public static HeadersBuilder<?> noContent()
public static BodyBuilder badRequest()
public static HeadersBuilder<?> notFound()
public static BodyBuilder unprocessableEntity()
public static BodyBuilder internalServerError()

public static <T> ResponseEntity<T> ok(@Nullable T body)
```
