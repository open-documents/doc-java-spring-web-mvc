	参考文档:P864

DispatcherServlet完成它的功能，需要使用到HandlerMapping、

在初始化DispatcherServlet的WebApplicationContext（该属性由DispatcherServlet的FrameworkServlet提供）时，可以自定义特殊bean的实例，但是名称必须使用在DispatcherServlet中定义好的默认名称（见特殊Bean类型的默认名称）。
当有自定义的特殊Bean实例时，DispatcherServlet在初始化它所需要的这些Bean实例时，会优先从WebApplicationContext中拿取自定义的特殊Bean实例，不然会通过SPI的机制拿取DispatcherServlet.properties文件定义好的特殊Bean类型的实例。

| bean类型                              | 描述 |
| ------------------------------------- | ---- |
| HandlerMapping                        |      |
| HandlerAdapter                        |      |
| HandlerExceptionResolver              |      |
| ViewResolver                          |      |
| LocaleResolver, LocaleContextResolver |      |
| MultipartResolver                     |      |
| FlashMapManager                       |      |
| ThemeResolver                         |      |

# 特殊Bean类型的默认名称

下面是已经在DispatcherServlet中定义好的默认名称。

| bean类型                              | 默认名称                                                                                |
| ------------------------------------- | --------------------------------------------------------------------------------------- |
| HandlerMapping                        | handlerMapping。只有当detectAllHandlerMappings为false时才会被使用。                     |
| HandlerAdapter                        | handlerAdapter。只有当detectAllHandlerAdapters为false时才会被使用。                     |
| HandlerExceptionResolver              | handlerExceptionResolver。只有当detectAllHandlerExceptionResolvers为false时才会被使用。 |
| ViewResolver                          | viewResolver。只有当detectAllViewResolvers为false时才会被使用。                         |
| LocaleResolver, LocaleContextResolver | localeResolver                                                                          |
| MultipartResolver                     | multipartResolver                                                                       |
| FlashMapManager                       |                                                                                         |
| ThemeResolver                         | themeResolver                                                                           |
|                                       | viewNameTranslator                                                                      |
|                                       | flashMapManager                                                                         |
|                                       |                                                                                         |

# 特殊Bean类型的默认实现类

定义在和DispatcherServlet位于同一文件夹的DispatcherServlet.properties文件中。
| Bean类型                              | 默认实现类                                                                                                                                                                                                                             |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HandlerMapping                        | org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,<br/>org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping,<br/> org.springframework.web.servlet.function.support.RouterFunctionMapping |
| HandlerAdapter                        | org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,<br/>org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,<br/>org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter,<br/>org.springframework.web.servlet.function.support.HandlerFunctionAdapter |
| HandlerExceptionResolver              | org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,<br/>org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,<br/>org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver  |
| ViewResolver                          | org.springframework.web.servlet.view.InternalResourceViewResolver                                                                                                                                                                |
| LocaleResolver, LocaleContextResolver | org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver                                                                                                                                                                        |
| RequestToViewNameTranslator           | org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator                                                                                                                                                                |
| FlashMapManager                       | org.springframework.web.servlet.support.SessionFlashMapManager                                                                                                                                                               |
| ThemeResolver                         | org.springframework.web.servlet.theme.FixedThemeResolver                                                                                                                                                                               |
