
# 一、注册

RequestMappingHandlerMapping通过下面的@Bean方法被注册进spring容器。
```java
/* --------------------------------- WebMvcConfigurationSupport ---------------------------------
 */
@Bean  
@SuppressWarnings("deprecation")  
public RequestMappingHandlerMapping requestMappingHandlerMapping(  
      @Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager,  
      @Qualifier("mvcConversionService") FormattingConversionService conversionService,  
      @Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider) {  

	// 简单地使用RequestMappingHandlerMapping()无参构造创建实例
    RequestMappingHandlerMapping mapping = createRequestMappingHandlerMapping();

    mapping.setOrder(0);  
    mapping.setInterceptors(getInterceptors(conversionService, resourceUrlProvider));  
    mapping.setContentNegotiationManager(contentNegotiationManager);  
    mapping.setCorsConfigurations(getCorsConfigurations());  

	
    PathMatchConfigurer pathConfig = getPathMatchConfigurer();  // 1

	// PathMatchConfigurer#getPatternParser(): 默认(初)值为null
    if (pathConfig.getPatternParser() != null) {  // 
        mapping.setPatternParser(pathConfig.getPatternParser());  
    }  
    else {  
        // 默认(初)值为null, 实例来源依次是:
        //   - PathMatchConfigurer#urlPathHelper不为null,则使用
        //   - PathMatchConfigurer#defaultUrlPathHelper不为null,则使用
        //   - new UrlPathHelper()
        mapping.setUrlPathHelper(pathConfig.getUrlPathHelperOrDefault());  
        // 默认(初)值为null, 实例来源依次是:
        //   - PathMatchConfigurer#pathMatcher不为null,则使用
        //   - PathMatchConfigurer#defaultPathMatcher不为null,则使用
        //   - new AntPathMatcher()
        mapping.setPathMatcher(pathConfig.getPathMatcherOrDefault());  

		// 1. 是否开启后缀".*"模式匹配,即匹配"/users",也就会匹配"/users.*""
		// 2. 默认(初)值为null, 相关方法--setUseSuffixPatternMatch(Boolean)(@Deprecated)
		// 3. @Deprecated,从5.2.4
        Boolean useSuffixPatternMatch = pathConfig.isUseSuffixPatternMatch();  
        if (useSuffixPatternMatch != null) {
            mapping.setUseSuffixPatternMatch(useSuffixPatternMatch);  
        }  
        // 1. 是否开启后缀模式匹配只用于匹配configureContentNegotiation()设置的后缀路径扩展名
        // 2. 默认(初)值为null, 相关方法--setUseRegisteredSuffixPatternMatch(Boolean)(@Deprecated)
		// 3. @Deprecated,从5.2.4
        Boolean useRegisteredSuffixPatternMatch = pathConfig.isUseRegisteredSuffixPatternMatch();  
        if (useRegisteredSuffixPatternMatch != null) {  
            mapping.setUseRegisteredSuffixPatternMatch(useRegisteredSuffixPatternMatch);  
        }  
    }  
    // 1. 是否使用路径末尾的破折号匹配,即匹配/users,也就会匹配/users/
	// 2. 默认(初)值为null, 相关方法--setUseTrailingSlashMatch(Boolean)
	// 3. @Deprecated
    Boolean useTrailingSlashMatch = pathConfig.isUseTrailingSlashMatch();  
    if (useTrailingSlashMatch != null) {  
        mapping.setUseTrailingSlashMatch(useTrailingSlashMatch);  
    }  
    // 1. 
	// 2. 默认(初)值为null, 相关方法--addPathPrefix(String,Predicate<Class<?>>),从5.1
	// 3. Map<String, Predicate<Class<?>>>
    if (pathConfig.getPathPrefixes() != null) {  
        mapping.setPathPrefixes(pathConfig.getPathPrefixes());  
    }
  
    return mapping;  
}
```

# 1）getPathMatchConfigurer()
```java
/* --------------------------------- WebMvcConfigurationSupport ---------------------------------
 */
protected PathMatchConfigurer getPathMatchConfigurer() {  
	// this.pathMatchConfigurer:
	//   - PathMatchConfigurer类型
	//   - 默认(初)值为null
    if (this.pathMatchConfigurer == null) {  
        this.pathMatchConfigurer = new PathMatchConfigurer();  
        // - 默认NoOp方法
        // - 父类DelegatingWebMvcConfiguration重写了
        configurePathMatch(this.pathMatchConfigurer);  
    }  
    return this.pathMatchConfigurer;  
}
/* --------------------------------- DelegatingWebMvcConfiguration ---------------------------------
 */
protected void configurePathMatch(PathMatchConfigurer configurer) {  
	// this.configurers:
	//   - WebMvcConfigurerComposite类型
	//   - 默认(初)值为 new WebMvcConfigurerComposite()
    this.configurers.configurePathMatch(configurer);  
}
/* --------------------------------- WebMvcConfigurerComposite ---------------------------------
 */
public void configurePathMatch(PathMatchConfigurer configurer) {  
	// this.delegates:
	//   - List<WebMvcConfigurer>类型
	//   - 默认(初)值为 new ArrayList<>()
    for (WebMvcConfigurer delegate : this.delegates) {  
        delegate.configurePathMatch(configurer);  
    }  
}
```

# 二、（重要）初始化

总体来说，做了2件事：
- 
```java
/* ----------------------------------- RequestMappingHandlerMapping -----------------------------------
 *
 */
public void afterPropertiesSet() {  

	// RequestMappingInfo.BuilderConfiguration this.config
    this.config = new RequestMappingInfo.BuilderConfiguration();

	// 1. this.useTrailingSlashMatch 默认(初)值为true
    this.config.setTrailingSlashMatch(useTrailingSlashMatch());  

	// 1. - getContentNegotiationManager() 返回this.contentNegotiationManager
	//    - this.contentNegotiationManager默认(初)值为 new ContentNegotiationManager()
    this.config.setContentNegotiationManager(getContentNegotiationManager());  

	// 1. 默认(初)值为null
	// 
    if (getPatternParser() != null) {  
        this.config.setPatternParser(getPatternParser());  
	    ... // assert
    }  
    else {  
	    // 1. - useSuffixPatternMatch(): 返回this.useSuffixPatternMatch
	    //    - this.useSuffixPatternMatch默认(初)值为false
        this.config.setSuffixPatternMatch(useSuffixPatternMatch());  
        // 1. - useRegisteredSuffixPatternMatch(): 返回this.useRegisteredSuffixPatternMatch
        //    - this.useRegisteredSuffixPatternMatch默认(初)值为false
        this.config.setRegisteredSuffixPatternMatch(useRegisteredSuffixPatternMatch());  
        // 1. - getPathMatcher(): 返回其父父父类AbstractHanlderMapping#pathMatcher
        //    - AbstractHanlderMapping#pathMatcher默认(初)值为 new AntPathMatcher()
        this.config.setPathMatcher(getPathMatcher());  
    }  

    super.afterPropertiesSet();  
}

/* --------------------------------- AbstractHandlerMethodMapping<T> ---------------------------------
 * 描述: 该类是RequestMappingHandlerMapping的父父类
 */
public void afterPropertiesSet() {  
    initHandlerMethods();  
}
protected void initHandlerMethods() {  
	// 获取容器中是Object.class及其子类的bean
    for (String beanName : getCandidateBeanNames()) {  
	    // SCOPED_TARGET_NAME_PREFIX = "scopedTarget."
	    if (!beanName.startsWith(SCOPED_TARGET_NAME_PREFIX)) {  
		    
            processCandidateBean(beanName);  
        }  
    }  
    // 不重要的方法
    // 
    handlerMethodsInitialized(getHandlerMethods());  
}
```
# 1）方法--getCandidateBeanNames()
```java
/* --------------------------------- AbstractHandlerMethodMapping<T> ---------------------------------
 * 描述: 该类是RequestMappingHandlerMapping的父父类
 */
protected String[] getCandidateBeanNames() {
	// 1. this.detectHandlerMethodsInAncestorContexts默认(初)值为false
    return (this.detectHandlerMethodsInAncestorContexts ? 
         BeanFactoryUtils.beanNamesForTypeIncludingAncestors(obtainApplicationContext(), Object.class) :  
         // 
         obtainApplicationContext().getBeanNamesForType(Object.class));  
}
```
# 2）方法--processCandidateBean()
```java
/* --------------------------------- AbstractHandlerMethodMapping<T> ---------------------------------
 * 描述: 该类是RequestMappingHandlerMapping的父父类
 */
protected void processCandidateBean(String beanName) {  
    Class<?> beanType = null;  
    try {  
	    // - 获取getBean()返回对象的类型
	    // - 注意:只是语义上的,并不是真正调用getBean()
        beanType = obtainApplicationContext().getType(beanName);  
    }  
	... // catch
    // isHandler(Class<?>): 
    //   - 抽象方法,RequestMappingHandlerMapping重写了该方法
    //   - 满足的条件(满足一个即可):
    //       1) 类上标注了@Controller注解
	//       2) 类上标注了@RequestMapping注解
    if (beanType != null && isHandler(beanType)) {  
        detectHandlerMethods(beanName);  
    }
}
/* --------------------------------- AbstractHandlerMethodMapping<T> ---------------------------------
 * 描述: 该类是RequestMappingHandlerMapping的父父类
 */
protected void detectHandlerMethods(Object handler) {  
    Class<?> handlerType = (handler instanceof String ?  
         obtainApplicationContext().getType((String) handler) : handler.getClass());  
  
    if (handlerType != null) {  
	    // 如果不是代理类,则简单地返回原类
	    // 如果是CGLIB代理类,则返回target class
        Class<?> userType = ClassUtils.getUserClass(handlerType);  
		Map<Method, T> methods = MethodIntrospector.selectMethods(userType,  
            (MethodIntrospector.MetadataLookup<T>) method -> {  
                try {  
                    return getMappingForMethod(method, userType);  
                }  
                ... // catch
            });  
		... //log
	    methods.forEach((method, mapping) -> {  
            Method invocableMethod = AopUtils.selectInvocableMethod(method, userType);  
            // 
            registerHandlerMethod(handler, invocableMethod, mapping);  
        });  
    }  
}
```

```java
/* --------------------------------- AbstractHandlerMethodMapping<T> ---------------------------------
 * 描述: 该类是RequestMappingHandlerMapping的父父类
 */
protected void registerHandlerMethod(Object handler, Method method, T mapping) {  
    this.mappingRegistry.register(mapping, handler, method);  
}
/* --------------------------------- MappingRegistry ---------------------------------
 */
public void register(T mapping, Object handler, Method method) {  
    this.readWriteLock.writeLock().lock();  
    try {  
        HandlerMethod handlerMethod = createHandlerMethod(handler, method);  
        validateMethodMapping(handlerMethod, mapping);  
  
        Set<String> directPaths = AbstractHandlerMethodMapping.this.getDirectPaths(mapping);  
        for (String path : directPaths) {  
            this.pathLookup.add(path, mapping);  
        }  
  
        String name = null;  
        // 
        if (getNamingStrategy() != null) {  
         name = getNamingStrategy().getName(handlerMethod, mapping);  
         addMappingName(name, handlerMethod);  
        }  
  
        CorsConfiguration corsConfig = initCorsConfiguration(handler, method, mapping);  
        if (corsConfig != null) {  
            corsConfig.validateAllowCredentials();  
            this.corsLookup.put(handlerMethod, corsConfig);  
        }  
  
        this.registry.put(mapping,  
            new MappingRegistration<>(mapping, handlerMethod, directPaths, name, corsConfig != null));  
    }  
    finally {  
        this.readWriteLock.writeLock().unlock();  
    }  
}
```









### 方法--MethodIntrospector.selectMethods()
```java
public static <T> Map<Method, T> selectMethods(Class<?> targetType, final MetadataLookup<T> metadataLookup) {  
   final Map<Method, T> methodMap = new LinkedHashMap<>();  
   Set<Class<?>> handlerTypes = new LinkedHashSet<>();  
   Class<?> specificHandlerType = null;  
  
   if (!Proxy.isProxyClass(targetType)) {  
      specificHandlerType = ClassUtils.getUserClass(targetType);  
      handlerTypes.add(specificHandlerType);  
   }  
   handlerTypes.addAll(ClassUtils.getAllInterfacesForClassAsSet(targetType));  
  
   for (Class<?> currentHandlerType : handlerTypes) {  
      final Class<?> targetClass = (specificHandlerType != null ? specificHandlerType : currentHandlerType);  
  
      ReflectionUtils.doWithMethods(currentHandlerType, method -> {  
         Method specificMethod = ClassUtils.getMostSpecificMethod(method, targetClass);  
         T result = metadataLookup.inspect(specificMethod);  
         if (result != null) {  
            Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(specificMethod);  
            if (bridgedMethod == specificMethod || metadataLookup.inspect(bridgedMethod) == null) {  
               methodMap.put(specificMethod, result);  
            }  
         }  
      }, ReflectionUtils.USER_DECLARED_METHODS);  
   }  
  
   return methodMap;  
}
```










# 3）重要--getMappingForMethod()
```java
/* ------------------------------- RequestMappingHandlerMapping -------------------------------

*/
protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {  
    RequestMappingInfo info = createRequestMappingInfo(method);  
    if (info != null) {  
        RequestMappingInfo typeInfo = createRequestMappingInfo(handlerType);  
        if (typeInfo != null) {  
            info = typeInfo.combine(info);  
        }  
        String prefix = getPathPrefix(handlerType);  
        if (prefix != null) {  
            info = RequestMappingInfo.paths(prefix).options(this.config).build().combine(info);  
        }  
    }  
    return info;  
}
```

```java
/* ------------------------------- RequestMappingHandlerMapping -------------------------------

*/
private RequestMappingInfo createRequestMappingInfo(AnnotatedElement element) {  
	
    RequestMapping requestMapping = 
	    AnnotatedElementUtils.findMergedAnnotation(element, RequestMapping.class);  

    RequestCondition<?> condition = (element instanceof Class ?  
		 // 默认返回null
         getCustomTypeCondition((Class<?>) element) : 
         // 默认返回null
         getCustomMethodCondition((Method) element));  
    return (requestMapping != null ? createRequestMappingInfo(requestMapping, condition) : null);  
}

/* ------------------------------- RequestMappingHandlerMapping -------------------------------

*/
protected RequestMappingInfo createRequestMappingInfo(  
      RequestMapping requestMapping, @Nullable RequestCondition<?> customCondition) {  
  
    RequestMappingInfo.Builder builder = RequestMappingInfo  
         .paths(resolveEmbeddedValuesInPatterns(requestMapping.path()))  // 返回类型DefaultBuilder
         .methods(requestMapping.method())  
         .params(requestMapping.params())  
         .headers(requestMapping.headers())  
         .consumes(requestMapping.consumes())  
         .produces(requestMapping.produces())  
         .mappingName(requestMapping.name());  
    if (customCondition != null) {  
        builder.customCondition(customCondition);  
    }  
    // this.config默认(初)值为 new RequestMappingInfo.BuilderConfiguration()
    return builder.options(this.config).build();  
}
```

```java
/* ------------------------------- RequestMappingHandlerMapping -------------------------------

*/
protected String[] resolveEmbeddedValuesInPatterns(String[] patterns) {  
	// 
    if (this.embeddedValueResolver == null) {  
        return patterns;  
    }  
    else {  
        String[] resolvedPatterns = new String[patterns.length];  
        for (int i = 0; i < patterns.length; i++) {  
            resolvedPatterns[i] = this.embeddedValueResolver.resolveStringValue(patterns[i]);  
        }  
        return resolvedPatterns;  
   }  
}
```

```java
/* ------------------------------------- DefaultBuilder -------------------------------------

*/
public RequestMappingInfo build() {  
  
	PathPatternsRequestCondition pathPatterns = null;  
	PatternsRequestCondition patterns = null;  

	// this.options 在RequestMappingHandlerMapping#createRequestMappingInfo()中赋值
	// this.options 值为new RequestMappingInfo.BuilderConfiguration()
	if (this.options.patternParser != null) {  // 默认不满足  
	    pathPatterns = (ObjectUtils.isEmpty(this.paths) ? 
		    //  EMPTY_PATH_PATTERNS = new PathPatternsRequestCondition()
            EMPTY_PATH_PATTERNS :  
            new PathPatternsRequestCondition(this.options.patternParser, this.paths));  
    }  
	else {  // 默认满足  
	    patterns = (ObjectUtils.isEmpty(this.paths) ?  
		    // EMPTY_PATTERNS = new PatternsRequestCondition()
	        EMPTY_PATTERNS :  
            new PatternsRequestCondition(  
                this.paths,
                // @Nullable UrlPathHelper urlPathHelper
                null,
                this.options.getPathMatcher(),          // 为null
                this.options.useSuffixPatternMatch(),   // 为false
                this.options.useTrailingSlashMatch(),   // 为true
                this.options.getFileExtensions()));     // @Deprecated
    }  

	// this.options.getContentNegotiationManager()为null
	ContentNegotiationManager manager = this.options.getContentNegotiationManager();  


	return new RequestMappingInfo(  
        this.mappingName,                      // @RequestMapping#name(), @Nullable
        pathPatterns, 
        patterns,  
        // RequestMethodsRequestCondition
        ObjectUtils.isEmpty(this.methods) ?
	        // EMPTY_REQUEST_METHODS = new RequestMethodsRequestCondition()
            EMPTY_REQUEST_METHODS : 
            new RequestMethodsRequestCondition(this.methods),  
        // ParamsRequestCondition
        ObjectUtils.isEmpty(this.params) ?     
	        // EMPTY_PARAMS = new ParamsRequestCondition()
            EMPTY_PARAMS : 
            new ParamsRequestCondition(this.params),  

        // 
        ObjectUtils.isEmpty(this.headers) ?  
	        // EMPTY_HEADERS = new HeadersRequestCondition()
            EMPTY_HEADERS : 
            new HeadersRequestCondition(this.headers),  

		
        ObjectUtils.isEmpty(this.consumes) && !this.hasContentType ?  
            EMPTY_CONSUMES : 
            new ConsumesRequestCondition(this.consumes, this.headers), 


        ObjectUtils.isEmpty(this.produces) && !this.hasAccept ?  
            EMPTY_PRODUCES : 
            new ProducesRequestCondition(this.produces, this.headers, manager),  

        this.customCondition != null ?  
            new RequestConditionHolder(this.customCondition) : 
            EMPTY_CUSTOM,  

        this.options);  
    }  
}
```



