	参考文档:P924

@ResponseEntity注解与@ResponseBody类似，但是有状态（status）和头（headers）。

# 例子

```java
@GetMapping("/something") 
public ResponseEntity<String> handle() {   
	String body = ... ;   
	String etag = ... ;   
	return ResponseEntity.ok().eTag(etag).body(body);
}
```




