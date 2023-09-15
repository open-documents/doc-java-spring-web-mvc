	参考文档:P993

```java
@Configuration
@EnableWebMvc 
public class WebConfig implements WebMvcConfigurer {   
	@Override   
	public void configureViewResolvers(ViewResolverRegistry registry) {   
		registry.freeMarker();
	}   
	// Configure FreeMarker...   
	@Bean   
	public FreeMarkerConfigurer freeMarkerConfigurer() { 
		FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
		configurer.setTemplateLoaderPath("/WEB-INF/freemarker"); 
		return configurer;
	}
}
```


# 使用xml配置

## 设置模板路径 
```xml
<mvc:annotation-driven/>
	<mvc:view-resolvers>
		<mvc:freemarker/>
	</mvc:view-resolvers>
<!-- Configure FreeMarker... -->
<mvc:freemarker-configurer>
	<mvc:template-loader-path location="/WEB-INF/freemarker"/>
</mvc:freemarker-configurer>
```

```xml
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer"> 
	<property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
</bean>
```

## 设置全局设置

## 设置共享变量

```xml
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">   
	<property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
	<property name="freemarkerVariables">
		<map>
			<entry key="xml_escape" value-ref="fmXmlEscape"/>
		</map>
	</property>
</bean>
<bean id="fmXmlEscape" class="freemarker.template.utility.XmlEscape"/>
```