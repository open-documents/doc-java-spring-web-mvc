

通过在控制器方法参数上标注@SessionAttribute注解来从Session对象中取得已存在的值。
```java
@Controller  
public class MyController {  
  
    @RequestMapping("/")  
    public String index(HttpServletRequest httpServletRequest,Model model){  
        HttpSession session = httpServletRequest.getSession();  
        session.setAttribute("student",new Student(6));  
        return "index";  
    }  
    @RequestMapping("/test")  
    public String test(@SessionAttribute("student") Student student){  
        System.out.println(student);  
        return "success";  
    }
}
```

如果打算添加或删除session属性，可以考虑在控制器参数中注入org.springframework.web.context.request.WebRequest
或jakarta.servlet.http.HttpSession