
@Controller标注在类上，表示这个类作为wen控制器组件。

@Controller支持自动检测和自动注册bean definition。要启用自动检测，需要使用包扫描。

# RestController

RestController是@Controller和@ResponseBody的合成注解。

表示每个方法都继承@ResponseBody注解，因此直接向response body中写入，而不会被ViewResolver解析和不会被HTML模板渲染。