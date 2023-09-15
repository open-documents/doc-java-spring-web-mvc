

```java
/* ---------------------------------- DispatcherServlet ----------------------------------
 *
 */

protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {  
    logRequest(request);  // 1
  
    // Keep a snapshot of the request attributes in case of an include,  
    // to be able to restore the original attributes after the include.   
    Map<String, Object> attributesSnapshot = null;  
    // 判断是否是Include请求
    if (WebUtils.isIncludeRequest(request)) {  // 是Include请求
        attributesSnapshot = new HashMap<>();  
        Enumeration<?> attrNames = request.getAttributeNames();  
        while (attrNames.hasMoreElements()) {  
            String attrName = (String) attrNames.nextElement();  
            // this.cleanupAfterInclude: 在Include请求完成后,决定是否移除请求中的属性
            // DEFAULT_STRATEGIES_PREFIX = "org.springframework.web.servlet"
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
	            // 将请求中以"org.springframework.web.servlet"开头的键对应的值赋值一份到快照中
	            attributesSnapshot.put(attrName, request.getAttribute(attrName));  
            }  
        }  
    }  
  
    // Make framework objects available to handlers and view objects.  

    // WEB_APPLICATION_CONTEXT_ATTRIBUTE = "org.springframework.web.servlet.DispatcherServlet.CONTEXT"
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());  
    // LOCALE_RESOLVER_ATTRIBUTE = "org.springframework.web.servlet.DispatcherServlet.LOCALE_RESOLVER"
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);  
    // THEME_RESOLVER_ATTRIBUTE = "org.springframework.web.servlet.DispatcherServlet.THEME_RESOLVER"
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);  
    // THEME_SOURCE_ATTRIBUTE = "org.springframework.web.servlet.DispatcherServlet.THEME_SOURCE"
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());  

	// 默认注入SessionFlashMapManager类型实例
    if (this.flashMapManager != null) {  
	    FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);  
	    if (inputFlashMap != null) {
		    // INPUT_FLASH_MAP_ATTRIBUTE = "org.springframework.web.servlet.DispatcherServlet.INPUT_FLASH_MAP"
            request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));  
        }  
        // OUTPUT_FLASH_MAP_ATTRIBUTE = "org.springframework.web.servlet.DispatcherServlet.OUTPUT_FLASH_MAP"
        request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());  
        // FLASH_MAP_MANAGER_ATTRIBUTE = "org.springframework.web.servlet.DispatcherServlet.FLASH_MAP_MANAGER"
        request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);  
    }  
  
    RequestPath previousRequestPath = null;  
    // 默认为false
    if (this.parseRequestPath) {  
	    // PATH_ATTRIBUTE = "org.springframework.web.util.ServletRequestPathUtils.PATH"
        previousRequestPath = (RequestPath) request.getAttribute(ServletRequestPathUtils.PATH_ATTRIBUTE);  
        ServletRequestPathUtils.parseAndCache(request);  
    }  
  
    try {  
        doDispatch(request, response);  
    }  
    finally {  
	    if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {  
            // Restore the original attribute snapshot, in case of an include.  
	        if (attributesSnapshot != null) {  
	            restoreAttributesAfterInclude(request, attributesSnapshot);  
            }  
        }  
        if (this.parseRequestPath) {  
            ServletRequestPathUtils.setParsedRequestPath(previousRequestPath, request);  
        }  
   }  
}
```

# 方法1 
```java
/* ---------------------------------- DispatcherServlet ----------------------------------
 *
 */
private void logRequest(HttpServletRequest request) {  
    LogFormatUtils.traceDebug(logger, traceOn -> {  
      String params;  
      if (StringUtils.startsWithIgnoreCase(request.getContentType(), "multipart/")) {  
         params = "multipart";  
      }  
      else if (isEnableLoggingRequestDetails()) {  
         params = request.getParameterMap().entrySet().stream()  
               .map(entry -> entry.getKey() + ":" + Arrays.toString(entry.getValue()))  
               .collect(Collectors.joining(", "));  
      }  
      else {  
         params = (request.getParameterMap().isEmpty() ? "" : "masked");  
      }  
  
      String queryString = request.getQueryString();  
      String queryClause = (StringUtils.hasLength(queryString) ? "?" + queryString : "");  
      String dispatchType = (!DispatcherType.REQUEST.equals(request.getDispatcherType()) ?  
            "\"" + request.getDispatcherType() + "\" dispatch for " : "");  
      String message = (dispatchType + request.getMethod() + " \"" + getRequestUri(request) +  
            queryClause + "\", parameters={" + params + "}");  
  
      if (traceOn) {  
         List<String> values = Collections.list(request.getHeaderNames());  
         String headers = values.size() > 0 ? "masked" : "";  
         if (isEnableLoggingRequestDetails()) {  
            headers = values.stream().map(name -> name + ":" + Collections.list(request.getHeaders(name)))  
                  .collect(Collectors.joining(", "));  
         }  
         return message + ", headers={" + headers + "} in DispatcherServlet '" + getServletName() + "'";  
      }  
      else {  
         return message;  
      }  
   });  
}
```