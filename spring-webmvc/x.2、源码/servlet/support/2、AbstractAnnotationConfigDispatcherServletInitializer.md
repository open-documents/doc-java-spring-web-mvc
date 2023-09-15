下面是定义：
```java
public abstract class AbstractAnnotationConfigDispatcherServletInitializer  
      extends AbstractDispatcherServletInitializer {
```
该类没有添加额外的功能，主要完成了父类部分方法的重写。

让我们看一下子类中被重写的方法。

# 被重写的方法

-- --
下面是来自父类AbstractContextLoaderInitializer的必须重写的方法：
## createRootApplicationContext()
```java
@Override  
@Nullable  
protected WebApplicationContext createRootApplicationContext() {  
	// 1 必须重写的方法--getRootConfigClasses
    Class<?>[] configClasses = getRootConfigClasses();  
    if (!ObjectUtils.isEmpty(configClasses)) {  
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
        context.register(configClasses);  
        return context;  
    }  
    else {  
        return null;  
    }  
}
```

-- --
下面是来自父类AbstractDispatcherServletInitializer的必须重写的方法。
## createServletApplicationContext()

```java
@Override  
protected WebApplicationContext createServletApplicationContext() {  
    AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();  
    // 2 必须被重写的方法--getServletConfigClasses
    Class<?>[] configClasses = getServletConfigClasses();  
    if (!ObjectUtils.isEmpty(configClasses)) {  
        context.register(configClasses);  
    }  
    return context;  
}
```


# 必须重写的方法


## 1.1）getRootConfigClasses

```java
@Nullable  
protected abstract Class<?>[] getRootConfigClasses();
```
在createRootApplicationContext()方法中被调用，返回值被注册到RootApplicationContext。

## 1.2）getServletConfigClasses

```java
@Nullable  
protected abstract Class<?>[] getServletConfigClasses();
```
在createServletApplicationContext()方法中被调用，返回值被注册到ServletApplicationContext。

-- --
## 2）来自父类AbstractDispatcherServletInitializer的方法

下面是来自父类AbstractDispatcherServletInitializer中必须重写的方法：
- getServletMappings()。[[1、AbstractDispatcherServletInitializer#1.2）getServletMappings()]]


# 可以重写的方法

## 来自父类AbstractContextLoaderInitializer的方法

下面是来自父类AbstractContextLoaderInitializer中可以重写的方法：
- getRootApplicationContextInitializers()

## 来自父类AbstractDispatcherServletInitializer的方法

下面是来自父类AbstractDispatcherServletInitializer中可以重写的方法：
- getServletApplicationContextInitializers()
- getServletFilters()
- customizeRegistration()
- createDispatcherServlet()（不推荐）
- createServletApplicationContext()

# 最佳实践

如果不需要ApplicationContext之间的层级关系，那么可以只通过getRootConfigClasses()返回配置类，而getServletConfigClasses()返回null。