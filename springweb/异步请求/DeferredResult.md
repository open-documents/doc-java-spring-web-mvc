	参考文档:P971

```java
@GetMapping("/quotes") 
@ResponseBody 
public DeferredResult<String> quotes() {   
	DeferredResult<String> deferredResult = new DeferredResult<String>();   
	// Save the deferredResult somewhere..   
	return deferredResult;
} 

// From some other thread... 
deferredResult.setResult(result);
```