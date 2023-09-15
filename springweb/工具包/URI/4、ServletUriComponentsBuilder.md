	参考文档:P965

```java
HttpServletRequest request = ...
// Re-uses scheme, host, port, path, and query string...
ServletUriComponentsBuilder.fromRequest(request)
// Re-uses scheme, host, port, and context path...
ServletUriComponentsBuilder.fromContextPath(request)
// Prepare a builder from the host, port, scheme, context path, and servlet mapping of the given HttpServletRequest.
ServletUriComponentsBuilder.fromServletMapping(request)

```