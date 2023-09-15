	参考文档:P877~P879

为了让Web应用支持国际化，需要识别每个用户的区域并根据这个区域显示内容。

在Spring MVC 中，<font color=44cf57>用户所在区域由区域解析器（Locale resovler）识别</font>，区域解析器必须实现LocaleResovler接口。

Springmvc采用的默认区域解析器是AcceptHeaderLocaleResolver。


# 接口定义

## LocaleResolver

## LocaleContextResolver



# 内置的LocaleResovler

Spring MVC 自带多个LocaleResovler实现，供不同的条件解析区域。

## AcceptHeaderLocaleResolver

它通过检验HTTP请求的accept-language头部来解析区域。这个头部是由用户的web浏览器根据底层操作系统的区域设置进行设定。
请注意，这个区域解析器无法改变用户的区域，因为它无法修改用户操作系统的区域设置。

## SessionLocaleResolver

Locale信息存储在session中，如果用户想要修改Locale信息，可以通过修改session中对应属性的值即可。

SessionLocaleResolver检验用户会话中预置的属性来解析区域。如果会话属性不存在，可以为这个解析器设置defaultLocale属性，如果没有指定默认Locale属性，它会根据accept-language HTTP头部确定默认区域。

## CookieLocaleResolver

Locale信息存储在Cookie中。

它通过检测浏览器中的Cookie来解析区域。如果Cookie不存在，可以为这个解析器设置defaultLocale属性，如果没有指定默认Locale属性，它会根据accept-language HTTP头部确定默认区域。

## FixedLocaleResolver

获得固定的Locale。

特点：
- 无法在运行时变更Locale。
- Locale一定有，TimeZone可选。
- 在初始化时指定Locale（与TimeZone）。


# 注册Locale Resolver

可以在Web应用的上下文中注册一个类型为LocaleResovler的bean定义区域解析器。

满足要求：
- bean的id必须是localResolver，便于DispatcherServlet自动发现。 
- 每个DispatcherServlet只能注册一个区域解析器。

```xml
<bean id="localeResolver" class="org.springframewrok.web.servlet.i18n.SessionLocaleResolver">
	<property name="defaultLocale" value="en"/>
</bean>
```