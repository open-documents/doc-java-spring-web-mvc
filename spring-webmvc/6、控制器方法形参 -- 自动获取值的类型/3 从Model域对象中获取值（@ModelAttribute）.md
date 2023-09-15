
see also:
- ModelFactory
- ModelAttributeMethodProcessor
- RequestMappingHandlerAdapter


@ModelAttribute可以标注方法上或方法参数上。这里是标注在方法参数上。

<font color=44cf57>控制器类没有添加@SessionAttributes注解，标注在形参上的@ModelAttribute总体流程如下：</font>
1）判断Model中是否有值。有值，取值，转到2；没有值，创建一个新的形参类型实例对象，转到2。
2）判断@ModelAttribute的binding属性。为true，绑定请求参数到取得的值中，转到3；为false，转到3。
3）判断形参是否进行@Valid校验（若添加有校验注解的话）。
4）结束。

@ModelAttribute注解在控制器类上标注@SessionAttributes注解后发生部分变化。


# 准备材料

```java
public class Student {  
    private int id;  
  
    public Student() {}  
    public Student(int id) {  
        this.id = id;  
    }  
    public void setId(int id) {  
        this.id = id;  
    }  
    @Override  
    public String toString() {  
        return "Student{" + "id=" + id + "}";  
    }  
}
```
# 例子

请求：**`/test?id=3`**
```java
@RequestMapping("/test")  
public String test(@ModelAttribute Student student,Model model){  
    System.out.println(model.getAttribute("student"));  
    return "success";  
}
// 输出
// Student{id=3}
```


# 默认行为

默认情况下，控制器方法参数类型不是简单类型（由BeanUtils#isSimpleProperty()决定）并且不被其他解析器解析时，这些控制器方法参数的@ModelAttribute可以标注。

请求：**`/test?id=2`**
```java
@RequestMapping("/test")  
public String test(Student student,Model model){  
    System.out.println(model.getAttribute("student"));  
    return "success";  
}
// 输出
// Student{id=2}
```
可以看到，Student类型参数在没有标注@ModelAttribute注解的情况下：
-  Model中没有该属性时，使用构造函数新建实例，并将请求参数绑定到实例对象中。
- 将该实例对象加入到Model中。

# binding属性

如果只想访问model，不希望绑定对象属性，设置binding属性为false。
```java
@ModelAttribute 
public AccountForm setUpForm() {   
	return new AccountForm();
}
@ModelAttribute 
public Account findAccount(@PathVariable String accountId) { 
	return accountRepository.findOne(accountId);
}
@PostMapping("update") 
public String update(@Valid AccountForm form, BindingResult result, @ModelAttribute(binding=false) Account account) {	①  
 	// ...
}
```

# 数据验证



# 数据绑定错误

数据绑定可能发生错误。默认情况下，会抛出BindException。在@ModelAttribute属性后跟上BindingResult类型参数，可以对控制器方法中出现的绑定错误进行检查。
```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit") 
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {
	if (result.hasErrors()) {
		return "petForm"; 
	} 
}
```


# x.1）源码解析 -- ModelAttributeMethodProcessor

```java
public final Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,  
      NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception { 

    String name = ModelFactory.getNameForParameter(parameter);  
    ModelAttribute ann = parameter.getParameterAnnotation(ModelAttribute.class);  
    if (ann != null) {  
       mavContainer.setBinding(name, ann.binding());  
    }  
   
    Object attribute = null;  
    BindingResult bindingResult = null;  
   
    if (mavContainer.containsAttribute(name)) {  
       attribute = mavContainer.getModel().get(name);  
    }  
    else {  
       // Create attribute instance  
       try {  
          attribute = createAttribute(name, parameter, binderFactory, webRequest);  
       }  
       catch (BindException ex) {  
          if (isBindExceptionRequired(parameter)) {  
             // No BindingResult parameter -> fail with BindException  
             throw ex;  
          }  
          // Otherwise, expose null/empty value and associated BindingResult  
          if (parameter.getParameterType() == Optional.class) {  
             attribute = Optional.empty();  
          }  
          else {  
             attribute = ex.getTarget();  
          }  
          bindingResult = ex.getBindingResult();  
       }  
    }  
   
   if (bindingResult == null) {  
      // Bean property binding and validation;  
      // skipped in case of binding failure on construction.      WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);  
      if (binder.getTarget() != null) {  
         if (!mavContainer.isBindingDisabled(name)) {  
            bindRequestParameters(binder, webRequest);  
         }  
         validateIfApplicable(binder, parameter);  
         if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {  
            throw new BindException(binder.getBindingResult());  
         }  
      }  
      // Value type adaptation, also covering java.util.Optional  
      if (!parameter.getParameterType().isInstance(attribute)) {  
         attribute = binder.convertIfNecessary(binder.getTarget(), parameter.getParameterType(), parameter);  
      }  
      bindingResult = binder.getBindingResult();  
   }  
  
    // Add resolved attribute and BindingResult at the end of the model  
    Map<String, Object> bindingResultModel = bindingResult.getModel();  
    mavContainer.removeAttributes(bindingResultModel);  
    mavContainer.addAllAttributes(bindingResultModel);  
   
    return attribute;  
}
```

# x.2）源码解析 -- ModelFactory

getNameForParameter() : MethodParameter


首先根据