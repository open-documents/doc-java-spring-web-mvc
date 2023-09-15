
当发送Http请求需要携带发送给服务端的请求参数时，根据请求类型的不同，携带的请求参数出现的位置也不同。

以下面的表单为例。
```xml
<form method="..." action="/test">
	<input type="text" name="param1"/>
	<input type="text" name="param2"/>
<form>
```
两者携带请求参数的格式<font color=44cf57>都</font>为：param1=xx1&param2=xx2  。

1）GET请求：请求参数是以 '?' 连接上述格式跟在请求URL路径后面（即 /test/param1=..&param2=..）。
2）POST请求：请求参数是写在请求体中。


# 默认行为



<font color=44cf57>当控制器方法形参名称和ServletRequest参数名称一致时，可以自动获取到该ServletRequest参数的值。</font>

以上述表单提交给如下的控制器请求方法为例。
```java
@Controller
public class MyController{
	@RequestMapping("/test")
	public String test(String param1, String param2){
		System.out.println(param1);  // 输出xx1, 表示获取到请求参数param1的值xx1
		System.out.println(param1);  // 输出xx2, 表示获取到请求参数param2的值xx2
		return ...;
	}
}
```

# @RequestParam

<font color=44cf57>当控制器方法形参名称与ServletRequest中参数名称不一致时，无法自动获取ServletRequest中该参数的值，此时需要使用@RequestParam来指定请求参数中参数的名称。</font>

以上述表单提交给如下的控制器请求方法为例。
```java
@Controller
public class MyController{
	@RequestMapping("/test")
	public String test(@RequestParam("param1")String var1, @RequestParam("param2")String var2){
		System.out.println(var1);  // 输出xx1, 表示获取到请求参数param1的值xx1
		System.out.println(var2);  // 输出xx2, 表示获取到请求参数param2的值xx2
		return ...;
	}
}
```

如果使用了@RequestParam注解，那么请求中必须携带@RequestParam注解指定名称的参数，否则会报错。

如果要在使用了@RequestParam注解的同时请求中不包含该参数的情况下不报错，可以让@RequestParam注解的required属性为false，或者将参数用Optional封装。

- 如果参数类型不是String，会自动进行类型转换。
- 如果参数类型是数组或list，该参数会获得多个相同请求参数名称的值。
- 如果参数类型是Map<String, String>或MultiValueMap<String, String>，并且@RequestParam注解没有指定名称，map会用请求参数名和请求参数值进行填充。

By default, any argument that is a simple value type (as determined by BeanUtils#isSimpleProperty) and is not resolved by any other argument resolver, is treated as if it were annotated with @RequestParam.


# 获取MultiPart

	参考文档:P918

在MultipartResolver被启用后，Context-type为multipart/form-data的POST请求内容会被解析，并能够作为常规请求参数获取。
```java
@Controller 
public class FileUploadController {   
	@PostMapping("/form")   
	public String handleFormUpload(@RequestParam("name") String name, 
								   @RequestParam("file") MultipartFile file) {  
		if (!file.isEmpty()) {   
			byte[] bytes = file.getBytes();
			// store the bytes somewhere   
			return "redirect:uploadSuccess"; 
		}
		return "redirect:uploadFailure";
	}
}
```
当声明参数类型为`List<MultipartFile>`，可以获取多个同名的文件内容。
当声明参数类型为`Map<String, MultipartFile>`或`MultiValueMap<String, MultipartFile>`，并且@RequestParam注解中没有指定名称，map按照 `名称-文件内容`的键值对填充。

## 数据绑定

multipart的内容也可以作为数据绑定内容的一部分。
```java
class MyForm {
	private String name; 
	private MultipartFile file;   
	// ...
} 
@Controller
public class FileUploadController {
	@PostMapping("/form")   
	public String handleFormUpload(MyForm form, BindingResult errors) { 
		if (!form.getFile().isEmpty()) {  
			byte[] bytes = form.getFile().getBytes();   
			// store the bytes somewhere   
			return "redirect:uploadSuccess"; 
		} 
		return "redirect:uploadFailure";
	}
}
```