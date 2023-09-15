# 视图解析器

下面是DispatcherServlet中的通过xml的方式配置WebApplication中的bean容器（DispatcherServlet中的WebApplication属性来源自其父类FrameworkServlet）。
```xml
<bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
	<property name="order" value="1"/>
	<property name="characterEncoding" value="UTF-8"/>
	<property name="templateEngine">
		<bean class="org.thymeleaf.spring5.SpringTemplateEngine">
			<property name="templateResolver">
				<bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
					<property name="prefix" value="WEB-INF/templates/"/>
					<property name="suffix" value=".html"/>
					<property name="templateMode" value="HTML5"/>
					<property name="characterEncoding" value="UTF-8"/>
				</bean>
			</property>
		</bean>
	</property>		
</bean>
```