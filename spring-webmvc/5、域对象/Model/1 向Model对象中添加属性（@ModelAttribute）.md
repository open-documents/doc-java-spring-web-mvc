标注在方法上的@ModelAttribute说明方法是用于添加一个或多个属性到model上。这样的方法能接受与@RequestMapping标注相同的参数类型，只不过不能直接被映射到具体的请求上。

在同一个控制器中，标注了@ModelAttribute的方法实际上会在@RequestMapping方法之前被调用。

@ModelAttribute标注方法有两种风格：

-   在第一种写法中，方法通过返回值的方式默认地将添加一个属性；
-   在第二种写法中，方法接收一个Model对象，然后可以向其中添加任意数量的属性。

以下是示例：
```java
// Add one attribute
// The return value of the method is added to the model under the name "account"
// You can customize the name via @ModelAttribute("myAccount")

@ModelAttribute
public Account addAccount(@RequestParam String number) {
	return new Account(number);
    // return accountManager.findAccount(number);
}

// Add multiple attributes

@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountManager.findAccount(number));
    // add more ...
}
```
一个控制器可以拥有多个@ModelAttribute方法。同个控制器内的所有这些方法，都会在@RequestMapping方法之前被调用。

@ModelAttribute方法也可以定义在@ControllerAdvice标注的类中，并且这些@ModelAttribute可以同时对许多控制器生效。
