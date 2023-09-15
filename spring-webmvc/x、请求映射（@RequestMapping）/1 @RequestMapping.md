	参考文档:P888

使用@RequestMapping注解将请求映射到controller方法。

# 一、定义

@RequestMapping注解提供通过URL、请求方法、请求参数、请求头、media类型来进行匹配的属性。

```java
@Target({ElementType.TYPE, ElementType.METHOD})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Mapping  
@Reflective(ControllerMappingReflectiveProcessor.class)  
public @interface RequestMapping {
	String name() default "";
	@AliasFor("path")
	String[] value() default {};
	@AliasFor("value")
	String[] path() default {};
	RequestMethod[] method() default {};
	String[] params() default {};
	String[] headers() default {};
	String[] consumes() default {};
	String[] produces() default {};
}
```

# 二、标注位置

标注在类上：表示类中所有方法都有共享的一个路径片段。
标注在方法上：表示该方法独有的路径匹配。

```java
@RestController 
@RequestMapping("/persons") 
class PersonController {
	@GetMapping("/{id}") 
	public Person getPerson(@PathVariable Long id) { 
		// ...
	}
	@PostMapping
	@ResponseStatus(HttpStatus.CREATED) 
	public void add(@RequestBody Person person) { 
		// ... 
	}
}
```



# 三、@RequestMapping注解的变体




# 支持的参数类型

	参考类型:P898

# 支持的返回类型

	参考文档:P900~P902


# 空值参数处理

	参考文档:P903

