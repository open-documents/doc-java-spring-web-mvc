
当请求的content-type为multipart/form-data或multipart/mixed stream时，可以通过@RequestPart获得完整的内容。

# 例子

有如下的请求头：
```text
POST /someUrl 
Content-Type: multipart/mixed 
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp 
Content-Disposition: form-data; name="meta-data" 
Content-Type: application/json; charset=UTF-8 
Content-Transfer-Encoding: 8bit 
{   
	"name": "value" 
} 
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp 
Content-Disposition: form-data; name="file-data"; filename="file.properties" 
Content-Type: text/xml 
Content-Transfer-Encoding: 8bit 
... File Data ...
```
需要获取“meta-data”的内容，可以通过@RequestParam，获取一个String类型的值。
但是如果想要获取完整json键值对，需要使用@RequestPart注解。
```java
@PostMapping("/") 
public String handle(@RequestPart("meta-data") MetaData metadata, 
					 @RequestPart("file-data") MultipartFile file) {}
```

# 验证

@RequestPart注解可以与jakarta.validation.Valid或spring的@Validated注解一起使用，两者都能触发bean验证。
默认情况下，验证错误会导致MethodArgumentNotValidException，会被抓换成400（BAD_REQUEST）。
可以通过Errors或BindingResult参数在控制器方法中处理验证错误。
```java
@PostMapping("/") 
public String handle(@Valid @RequestPart("meta-data") MetaData metadata, BindingResult result) {}
```


# 和@RequestParam的区别

@RequestParam 完成String转换成其他类型时，使用Converter。
@RequestPart使用HttpMessageConverter。





在MultipartResolver启用后，请求头为multipart/form-data的POST请求中的内容会被解析并能够被获取。

# 导入依赖

```xml
<dependency>  
    <groupId>commons-fileupload</groupId>  
    <artifactId>commons-fileupload</artifactId>  
    <version>1.5</version>  
</dependency>
```

# 配置MultipartResolver

在DispathcerServlet的ServletWebApplicationContext中添加multipartResolver Bean。
```xml
<!-- 配置MultipartResolver 用于文件上传 使用spring的CommosMultipartResolver -->  
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">  
    <property name="defaultEncoding" value="UTF-8"/>  
    <property name="maxUploadSize" value="5400000"/>  
    <!--  
    <property name="uploadTempDir" value="file"/>    --></bean>
```

# 上传单个文件

因为上传的文件多数要通过http继续访问，因此需要将其放在webapp文件夹下。
```java
@RequestMapping("/file")  
public String file(@RequestParam("file") MultipartFile file,HttpServletRequest request) throws IOException {  
    // 判断文件是否为空  
    if (!file.isEmpty()) {  
        try {  
            // 文件保存路径  
            System.out.println("真实路径:"+request.getSession().getServletContext().getRealPath("/"));  
            // 真实路径:D:\Workspace\IDEA\frame_spring\learning_springmvc\target\springmvc_learning-1.0-SNAPSHOT\
            System.out.println("file.getOriginalFilename():"+file.getOriginalFilename());  
            String filePath = request.getSession().getServletContext().getRealPath("/") + "upload/"  
                    + file.getOriginalFilename();  
            // 转存文件  
            file.transferTo(new File(filePath));  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
    return "success";  
}
```

# 上传多个同名文件

当上传多个同名文件时，可以用 `List<MultipartFile>` 来接收。

# 上传多个不同名文件

当上传多个不同名文件时，可以用 `Map<String, MultipartFile>` 或 `MultiValueMap<String, MultipartFile>` 来接收。

# multipart内容用于数据绑定

multipart的内容除了可以用上述的方式进行接收，还可以用于对象的绑定。
```java
class MyForm {   
	private String name;   
	private MultipartFile file;   
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


# x）MultipartFile

