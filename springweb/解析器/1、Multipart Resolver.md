	参考文档:P880、P918

- MultipartResolver用于解析multipart请求，如文件上传。
- MultipartResolver位于`org.springframework.web.multipart包`。
- MultipartResolver有一个基于容器的实现StandardServletMultipartResolver。

当一个请求有下面两个特征时，会用到Multipart Resolver：
- Post请求。
- content type内容：multipart/form-data。

Multipart Resolver会将HttpServletRequest包装成MultipartHttpServletRequest。

# 一、启用multipart处理

	参考文档:P881

为了开启multipart处理功能，需要在DispatcherServlet声明一个名称为multipartResolver的MultipartResolver bean。

## MultipartConfigElement

Servlet 3.0环境下启用MultiParsing，Servlet中设置`javax.servlet.MultipartConfigElement`。
MultipartConfigElement是`javax.servlet.annotation.MultipartConfig注解`的表示。
```java
public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer { 
	// ...  
	@Override
	protected void customizeRegistration(ServletRegistration.Dynamic registration) {
		registration.setMultipartConfig(getMultipartConfigElement());
	}
	private MultipartConfigElement getMultipartConfigElement() {
		MultipartConfigElement multipartConfigElement = new MultipartConfigElement( LOCATION, MAX_FILE_SIZE, MAX_REQUEST_SIZE, FILE_SIZE_THRESHOLD); 
		return multipartConfigElement;
	}
	// Temporary location where files will be stored
	private static final String LOCATION = "C:/temp/"; 
	// 5MB : Max file size. 
	// Beyond that size spring will throw exception.  
	private static final long MAX_FILE_SIZE = 5242880; 
	// 20MB : Total request size containing Multi part. 
	private static final long MAX_REQUEST_SIZE = 20971520; 
	// Size threshold after which files will be written to disk
	private static final int FILE_SIZE_THRESHOLD = 0; 
}
```
-- --
## @MultipartConfig

Servlet 3.0环境下启用MultiParsing，可以使用@MultipartConfig注解。
```java
@MultipartConfig(location=/tmp, 
				 fileSizeThreshold=0, 
				 maxFileSize=5242880, // 5 MB 
				 maxRequestSize=20971520) // 20 MB 
```

## xml方式
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    id="WebApp_ID" version="3.0">

    ····
    <servlet>
        <servlet-name>fileUpload</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>  
            <param-name>contextConfigLocation</param-name>  
            <param-value>classpath:fileUpload-servlet.xml</param-value>  
        </init-param>  
        <load-on-startup>0</load-on-startup>
        <multipart-config>
            <!--临时文件的目录-->
            <location>d:/</location>
            <!-- 上传文件最大2M -->
            <max-file-size>2097152</max-file-size>
            <!-- 上传文件整个请求不超过4M -->
            <max-request-size>4194304</max-request-size>
        </multipart-config>
    </servlet>
    ....
</web-app>
```




# StandardServletMultipartResolver

StandardServletMultipartResolver基于 Servlet 3.0 javax.servlet.http.Part API的MultipartResolver 接口的实现类。


# 文件上传例子

```java
import java.io.File;
import java.io.IOException;
import java.io.InputStream;

import javax.servlet.ServletException;
import javax.servlet.annotation.MultipartConfig;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.Part;

@WebServlet(urlPatterns = {"/upload.do"})
@MultipartConfig
public class FileUoloadServlet extends HttpServlet {
	private static final long serialVersionUID = 1873854450097318583L;
	@Override
	protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		//将发送请求页面中form表单里所有具有name属性的表单对象获取，获取文本
		String name1 = req.getParameter("name1");
		
		//得到fileName的部分
		Part part = req.getPart("fileName");
		long size = part.getSize();		            //上传的文件大小
		InputStream in = part.getInputStream();		//获取一个输入流
		//请求，当前应用的作用域中获取img的真实路径
		String realPath = req.getServletContext().getRealPath("img");
		String name = part.getName();		        //form表单中 file 域的名称
		String header = part.getHeader("Content-Disposition");		//文件的真实名称在请求头里
		//form-data; name="fileName"; filename="2.jpg"
		//获取文件的真实名称
		String realName = getFileRealName(part);
		//   File.separator  路径的分隔符
		String path = realPath+File.separator+realName;
		System.out.println(path);
		// 进行文件存储
		part.write(path);	
	}

	private String getFileRealName(Part part) {
		//form-data; name="fileName"; filename="""2.jpg"
				//文件真实名称在头信息中   获取指定的头信息
				String header = part.getHeader("Content-Disposition");
				String[] str = null;
				if(header != null) {
					str = header.split(";");
				}
				String filename = str[2];
				//找到第一个=  号的下标
				int index1 = filename.indexOf("=");
				//从第一个  = 号 开始截取
				filename = filename.substring(index1+2,filename.length()-1);
				return filename;
			}
}

```
表单部分：
```xml
<form action="upload.do" method="post" enctype="multipart/form-data" >
	<label>文本：</label><input type="text" name="name1" /><br>
	<label>文本：</label><input type="file" name="fileName" /><br>
	<input type="submit" value="提交" />
</form>
```


```java
@WebServlet(name = "fileUploadServlet", urlPatterns = {"/upload"}) 
@MultipartConfig(location=/tmp, 
				 fileSizeThreshold=0, 
				 maxFileSize=5242880, // 5 MB 
				 maxRequestSize=20971520) // 20 MB 
public class FileUploadServlet extends HttpServlet { 
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException { 
		//handle file upload
	}
}	
```


```java
@Controller
@RequestMapping(value = "/file2")
public class FileController2 {
    @RequestMapping(value = "/commUploadB")
    @ResponseBody
    public JSONObject commUploadB(MultipartHttpServletRequest request) {
        JSONObject json = new JSONObject();
        json.put("succ", false);
        try {
            MultipartFile file = request.getFile("uploadFileB");// 与页面input的name相同
            File imageFile = new File("d:/upload2.jpg");// 上传后的文件保存目录及名字
            file.transferTo(imageFile);// 将上传文件保存到相应位置
            json.put("succ", true);
            return json;
        } catch (Exception e) {
            e.printStackTrace();
            return json;
        }
    }
    @RequestMapping(value = "/commUploadC")
    @ResponseBody
    public JSONObject commUploadC(@RequestPart("uploadFileC") MultipartFile uploadFileC) {
        JSONObject json = new JSONObject();
        json.put("succ", false);
        try {
            File imageFile = new File("d:/upload3.jpg");// 上传后的文件保存目录及名字
            uploadFileC.transferTo(imageFile);// 将上传文件保存到相应位置
            json.put("succ", true);
            return json;
        } catch (Exception e) {
            e.printStackTrace();
            return json;
        }
    }
}
```