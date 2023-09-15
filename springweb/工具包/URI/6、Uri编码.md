	参考文档:P962


```java
URI uri = UriComponentsBuilder
	.fromPath("/hotel list/{city}")   
	.queryParam("q", "{q}")   
	.encode()   
	.buildAndExpand("New York", "foo+bar")   
	.toUri();
// Result is "/hotel%20list/New%20York?q=foo%2Bbar"
```