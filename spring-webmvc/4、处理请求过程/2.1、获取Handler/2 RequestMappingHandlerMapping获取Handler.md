
```java
/* -------------------------------- AbstractHandlerMethodMapping --------------------------------
 * 描述:
 *   该类是RequestMappingHandlerMapping的父父类
 */
protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
	// 该方法由父类AbstractHandlerMapping提供
    String lookupPath = initLookupPath(request);  
    this.mappingRegistry.acquireReadLock();  
    try {  
        HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);  
        return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);  
    }  
    finally {  
        this.mappingRegistry.releaseReadLock();  
    }  
}
```

# 一、初始化需要匹配的路径
```java
/* ------------------------------ AbstractHandlerMapping ------------------------------
 * 
 */
protected String initLookupPath(HttpServletRequest request) {  
	// 检查是否有PathPatternParser实例
    if (usesPathPatterns()) {  // AbstractHandlerMapping实例对象中有PathPatternParser实例对象
	    // UrlPathHelper.PATH_ATTRIBUTE = "org.springframework.web.util.UrlPathHelper.PATH"
        request.removeAttribute(UrlPathHelper.PATH_ATTRIBUTE);  
        // 直接从request中获取键为"org.springframework.web.util.ServletRequestPathUtils.PATH"的RequestPath属性
        RequestPath requestPath = ServletRequestPathUtils.getParsedRequestPath(request);  
        // 
        String lookupPath = requestPath.pathWithinApplication().value();  
        return UrlPathHelper.defaultInstance.removeSemicolonContent(lookupPath);  
    }  
    else {  
	    // spring5.x默认走这
        return getUrlPathHelper().resolveAndCacheLookupPath(request);  
    }  
}
```
# 二、获取相匹配的HandlerMethod

```java
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception { 
    List<Match> matches = new ArrayList<>();  
    // 
    List<T> directPathMatches = this.mappingRegistry.getMappingsByDirectPath(lookupPath);  
    if (directPathMatches != null) {  
        addMatchingMappings(directPathMatches, matches, request);  
    }  
    if (matches.isEmpty()) {  
        addMatchingMappings(this.mappingRegistry.getRegistrations().keySet(), matches, request);  
    }  
   if (!matches.isEmpty()) {  
      Match bestMatch = matches.get(0);  
      if (matches.size() > 1) {  
         Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));  
         matches.sort(comparator);  
         bestMatch = matches.get(0);  
         if (logger.isTraceEnabled()) {  
            logger.trace(matches.size() + " matching mappings: " + matches);  
         }  
         if (CorsUtils.isPreFlightRequest(request)) {  
            for (Match match : matches) {  
               if (match.hasCorsConfig()) {  
                  return PREFLIGHT_AMBIGUOUS_MATCH;  
               }  
            }  
         }  
         else {  
            Match secondBestMatch = matches.get(1);  
            if (comparator.compare(bestMatch, secondBestMatch) == 0) {  
               Method m1 = bestMatch.getHandlerMethod().getMethod();  
               Method m2 = secondBestMatch.getHandlerMethod().getMethod();  
               String uri = request.getRequestURI();  
               throw new IllegalStateException(  
                     "Ambiguous handler methods mapped for '" + uri + "': {" + m1 + ", " + m2 + "}");  
            }  
         }  
      }  
      request.setAttribute(BEST_MATCHING_HANDLER_ATTRIBUTE, bestMatch.getHandlerMethod());  
      handleMatch(bestMatch.mapping, lookupPath, request);  
      return bestMatch.getHandlerMethod();  
   }  
   else {  
      return handleNoMatch(this.mappingRegistry.getRegistrations().keySet(), lookupPath, request);  
   }  
}
```