	参考文档:P913-P915

默认情况下Spring MVC将模型中的数据存储到request域中。当一个请求结束后，数据就失效了。如果要跨页面使用。那么需要使用到session。

@SessionAttributes注解标注在控制器类上。
当被标注@SessionAttributes注解控制器方法向Model中添加属性时（@ModelAttribute），也向session域中添加一份。

# @SessionAttributes参数

| 属性  | 描述                                                      |
| ----- | --------------------------------------------------------- |
| names | 一个字符串数组。里面应写需要存储到session中数据的名称。和types是并集关系。   |
| value| names的别名。 |
| types |  根据指定参数的类型，将模型中对应类型的参数存储到session中。和names是并集关系。  |

只有指定的names和types实例在被添加进Model时才会被添加进Session域对象中，其他的都不会被添加进Session对象中。

# 例子

先请求：**`/`**
再请求：**`/test`**
```java
@Controller  
@SessionAttributes(types= {Student.class})
public class MyController {
	@RequestMapping("/")  
	public String index(HttpServletRequest httpServletRequest){  
	    httpServletRequest.getSession();  
	    return "index";  
	}
	@RequestMapping("/test")  
	public String test(@ModelAttribute("student") Student student, Model model, HttpServletRequest httpServletRequest){  
	    System.out.println(model.getAttribute("student"));  
	    System.out.println(httpServletRequest.getSession().getAttribute("student"));  
	    return "success";  
	}  
	@ModelAttribute  
	public void test2(Model model, HttpServletRequest httpServletRequest){  
	    model.addAttribute("student",new Student(5));  
	    System.out.println(httpServletRequest.getSession().getAttribute("student"));  
	}
}
```
结果：
```java
// 结果
// Student{id=5}  这是第一次从session对象中取值
// Student{id=5}  这是从Model对象中取值
// Student{id=5}  这是第二次从session对象中取值
```

# @SessionAttributes对标注在形参上@ModelAttribute功能的部分改变

当在控制器类上添加@SessionAttributes注解后，形参上标注@ModelAttribute后，@ModelAttribute注解功能发生了细微变化：
- 添加@SessionAttributes注解前，标注了@ModelAttribute的形参会尝试从Model中取值，如果Model中没有相应的值，那么会新建一个形参对应类型实例对象并完成数据绑定，然后将该实例对象加入到Model中。
- 添加@SessionAttributes注解后，标注了@ModelAttribute的形参不仅会尝试从Model中取值，还会尝试从Session对象中取值，当两者都不存在值时返回500，而不再创建新的实例对象。
- 添加@SessionAttributes注解后，如果Session域对象中有值，取得值后，会将该值添加到Model中。

<font color=44cf57>控制器类添加@SessionAttributes注解后，标注在形参上的@ModelAttribute总体流程如下：</font>
1）判断Model中是否有值。有就从Model中取值，转到3；没有，转到2。
2）判断Session中是否有值。有就从Session中取值，并将值添加到Model中，转到3；没有，抛出异常转到5。
3）判断@ModelAttribute的binding属性是否为false。为true，绑定请求参数到取得的值中，转到5；为false，转到5。
4）判断形参是否进行@Valid校验（若添加有校验注解的话）。
5）结束。