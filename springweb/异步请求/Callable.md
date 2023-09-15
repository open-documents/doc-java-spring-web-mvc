	参考文档:P971

```java
@PostMapping 
public Callable<String> processUpload(final MultipartFile file) {   
	return () -> "someView";
}
```