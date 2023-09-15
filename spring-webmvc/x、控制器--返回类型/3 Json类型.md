	参考文档:P925

Springmvc内置提供对Jackson序列化视图的支持。

在控制器方法上与和@ResponseBody注解或ResponEntity类配合使用@JsonView注解，来激活一个序列化视图。

@JsonView注解能够标注在需要序列化属性的getter方法上。

# 例子

控制器：
```java
@RestController
public class UserController {
	@GetMapping("/user")   
	@JsonView(User.WithoutPasswordView.class)   
	public User getUser() {   
		return new User("eric", "7!jd#h23"); 
	}
}
```
实体类：
```java
public class User {
	public interface WithoutPasswordView {};
	public interface WithPasswordView extends WithoutPasswordView {};
	private String username;
	private String password; 
	public User() {  } 
	public User(String username, String password) {
		this.username = username; 
		this.password = password; 
	}
	@JsonView(WithoutPasswordView.class)
	public String getUsername() { 
		return this.username;
	}
	@JsonView(WithPasswordView.class) 
	public String getPassword() { 
		return this.password;
	}
}
```


@JsonView注解的值可以是一个View Class的数组，但是每个控制器方法只能指定一个这样的@JsonView注解。

如果想要激活多个ViewClass，需要使用合成接口。

如果想要编程式地替换@JsonView注解，用MappingJacksonValue来包装返回值，并且为它来提供一个序列化视图。

```java
@RestController 
public class UserController {   
	@GetMapping("/user")   
	public MappingJacksonValue getUser() {
		User user = new User("eric", "7!jd#h23"); 
		MappingJacksonValue value = new MappingJacksonValue(user); 
		value.setSerializationView(User.WithoutPasswordView.class);  
		return value; 
	}
}
```

如果控制器依赖于视图解析，可以为model添加一个视图类属性。
```java
@Controller 
public class UserController extends AbstractController {   
	@GetMapping("/user")   
	public String getUser(Model model) { 
		model.addAttribute("user", new User("eric", "7!jd#h23"));
		model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class); 
		return "userView";
	}
}
```