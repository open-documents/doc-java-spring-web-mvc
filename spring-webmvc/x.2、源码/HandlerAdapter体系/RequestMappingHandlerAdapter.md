# # 初始化过程

因为 RequestMappingHandlerAdapter 实现了 InitializingBean 接口，因此 afterPropertiesSet 方法会被自动调用。

在 afterPropertiesSet 方法中，完成了各种参数解析器的初始化：
```java
@Override  
public void afterPropertiesSet() {  
    // Do this first, it may add ResponseBody advice beans  
    initControllerAdviceCache();  
   
    if (this.argumentResolvers == null) {  
       List<HandlerMethodArgumentResolver> resolvers = getDefaultArgumentResolvers();  
       this.argumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);  
    }  
    if (this.initBinderArgumentResolvers == null) {  
       List<HandlerMethodArgumentResolver> resolvers = getDefaultInitBinderArgumentResolvers();  
       this.initBinderArgumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);  
    }  
    if (this.returnValueHandlers == null) {  
       List<HandlerMethodReturnValueHandler> handlers = getDefaultReturnValueHandlers();  
       this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite().addHandlers(handlers);  
    }  
}
```
  
```java
private void initControllerAdviceCache() {  
    if (getApplicationContext() == null) {  
       return;  
    }  
  
    List<ControllerAdviceBean> adviceBeans = ControllerAdviceBean.findAnnotatedBeans(getApplicationContext());  
  
    List<Object> requestResponseBodyAdviceBeans = new ArrayList<>();  
  
   for (ControllerAdviceBean adviceBean : adviceBeans) {  
      Class<?> beanType = adviceBean.getBeanType();  
      if (beanType == null) {  
         throw new IllegalStateException("Unresolvable type for ControllerAdviceBean: " + adviceBean);  
      }  
      Set<Method> attrMethods = MethodIntrospector.selectMethods(beanType, MODEL_ATTRIBUTE_METHODS);  
      if (!attrMethods.isEmpty()) {  
         this.modelAttributeAdviceCache.put(adviceBean, attrMethods);  
      }  
      Set<Method> binderMethods = MethodIntrospector.selectMethods(beanType, INIT_BINDER_METHODS);  
      if (!binderMethods.isEmpty()) {  
         this.initBinderAdviceCache.put(adviceBean, binderMethods);  
      }  
      if (RequestBodyAdvice.class.isAssignableFrom(beanType) || ResponseBodyAdvice.class.isAssignableFrom(beanType)) {  
         requestResponseBodyAdviceBeans.add(adviceBean);  
      }  
   }  
  
   if (!requestResponseBodyAdviceBeans.isEmpty()) {  
      this.requestResponseBodyAdvice.addAll(0, requestResponseBodyAdviceBeans);  
   }  
  
   if (logger.isDebugEnabled()) {  
      int modelSize = this.modelAttributeAdviceCache.size();  
      int binderSize = this.initBinderAdviceCache.size();  
      int reqCount = getBodyAdviceCount(RequestBodyAdvice.class);  
      int resCount = getBodyAdviceCount(ResponseBodyAdvice.class);  
      if (modelSize == 0 && binderSize == 0 && reqCount == 0 && resCount == 0) {  
         logger.debug("ControllerAdvice beans: none");  
      }  
      else {  
         logger.debug("ControllerAdvice beans: " + modelSize + " @ModelAttribute, " + binderSize +  
               " @InitBinder, " + reqCount + " RequestBodyAdvice, " + resCount + " ResponseBodyAdvice");  
      }  
   }  
}
```
