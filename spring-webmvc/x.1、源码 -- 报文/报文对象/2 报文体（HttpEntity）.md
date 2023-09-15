HttpEntity代表请求报文或者响应报文实体，包含了报文头和报文体。

下面是定义：
```java
public class HttpEntity<T> {
	private final HttpHeaders headers;  
	@Nullable  
	private final T body;
}
```