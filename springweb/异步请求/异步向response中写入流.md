	参考文档:P976

设置Controller方法返回类型为StreamingResponseBody，可以绕过HttpMessageConverter直接向Response的outputStream中写入流。

```java
@GetMapping("/download") 
public StreamingResponseBody handle() {   
	return new StreamingResponseBody() {   
		@Override   
		public void writeTo(OutputStream outputStream) throws IOException {   
			// write...  
		}
	};
}
```

