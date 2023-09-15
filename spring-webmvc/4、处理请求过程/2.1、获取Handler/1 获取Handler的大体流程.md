
每个HandlerMapping实现类获取Handler的具体方法不一样，但是下面的方法是共同的。
```java
/* ---------------------------------- DispatcherServlet ----------------------------------
 *
 */
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {  
	// 默认有下面3个HandlerMapping:
	//   - BeanNameUrlHandlerMapping
	//   - RequestMappingHandlerMapping
	//   - RouterFunctionMapping
    if (this.handlerMappings != null) {  
        for (HandlerMapping mapping : this.handlerMappings) {  
            HandlerExecutionChain handler = mapping.getHandler(request);  
		    if (handler != null) {  
	            return handler;  
            }  
        }  
    }  
    return null;  
}
/* ------------------------------ AbstractHandlerMapping ------------------------------
 * 
 */
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {  
	// 抽象方法
    Object handler = getHandlerInternal(request);  
    if (handler == null) {  
        handler = getDefaultHandler();  
    }  
    if (handler == null) {  
        return null;  
    }  
    // Bean name or resolved handler?  
    if (handler instanceof String) {  
        String handlerName = (String) handler;  
        handler = obtainApplicationContext().getBean(handlerName);  
    }  
  
   // Ensure presence of cached lookupPath for interceptors and others  
   if (!ServletRequestPathUtils.hasCachedPath(request)) {  
      initLookupPath(request);  
   }  
  
   HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);  
  
   if (logger.isTraceEnabled()) {  
      logger.trace("Mapped to " + handler);  
   }  
   else if (logger.isDebugEnabled() && !DispatcherType.ASYNC.equals(request.getDispatcherType())) {  
      logger.debug("Mapped to " + executionChain.getHandler());  
   }  
  
   if (hasCorsConfigurationSource(handler) || CorsUtils.isPreFlightRequest(request)) {  
      CorsConfiguration config = getCorsConfiguration(handler, request);  
      if (getCorsConfigurationSource() != null) {  
         CorsConfiguration globalConfig = getCorsConfigurationSource().getCorsConfiguration(request);  
         config = (globalConfig != null ? globalConfig.combine(config) : config);  
      }  
      if (config != null) {  
         config.validateAllowCredentials();  
      }  
      executionChain = getCorsHandlerExecutionChain(request, executionChain, config);  
   }  
  
   return executionChain;  
}

```
