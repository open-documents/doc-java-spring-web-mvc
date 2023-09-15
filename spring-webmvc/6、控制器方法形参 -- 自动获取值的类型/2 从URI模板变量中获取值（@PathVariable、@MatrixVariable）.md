
通过PathPattern进行路径匹配时，如果使用`{pathVariable}`匹配一个路径片段后，通过@PathVariable注解将变量传入到java参数中。

举个例子：
```java
@GetMapping("/owners/{ownerId}/pets/{petId}") 
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
	// ...
}
```

```java
@Controller 
@RequestMapping("/owners/{ownerId}")
public class OwnerController { 
	@GetMapping("/pets/{petId}") 
	public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {  
		// ... 
	}
}
```

```java
@GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String name, @PathVariable String version, @PathVariable String ext) {   
	// ...
}
```


# 类型转换

	参考文档:P902

URI变量自动转换成合适的类型，或者抛出TypeMismatchException。
默认支持简单类型（如int，long，Date等）的转换，转换成其他类型见类型转换和DataBinder。 

# 指定名称

可以显示地指定@PathVariable的名称，如`@PathVariable("customId")`。当pathVariable和参数名称相同，并且编译带有-parameters参数时可以省略名称。


# 二、@MatrixVariable


矩阵变量能够出现在任意一个路径片段上。
每个变量之间用分号隔开；每个变量的多个值用逗号隔开。如`/cars;color=red,green;year=2012`。
同个变量多个值时也可以使用相同的变量名。如`color=red;color=green;color=blue`。

## 常见使用

如果URL中预计出现矩阵变量，那么必须使用一个URL变量来接收变量内容，以确保矩阵变量的正确匹配。
```java
// GET /pets/42;q=11;r=22 
@GetMapping("/pets/{petId}") 
public void findPet(@PathVariable String petId, @MatrixVariable int q) {
	// petId == 42   
	// q == 11
}
```
当多个路径片段可能出现矩阵变量，需要按如下方式消除歧义：
```java
// GET /owners/42;q=11/pets/21;q=22 
@GetMapping("/owners/{ownerId}/pets/{petId}") 
public void findPet(@MatrixVariable(name="q", pathVar="ownerId") int q1,   
					@MatrixVariable(name="q", pathVar="petId") int q2) {
		// q1 == 11   
		// q2 == 22
}
```
定义可选矩阵变量和默认值：
```java
// GET /pets/42 
@GetMapping("/pets/{petId}") 
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {
	// q == 1
}
```
使用MultiValueMap获取所有矩阵变量：
```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23 
@GetMapping("/owners/{ownerId}/pets/{petId}") 
public void findPet(@MatrixVariable MultiValueMap<String, String> matrixVars,   
					@MatrixVariable(pathVar="petId")MultiValueMap<String, String> petMatrixVars) {
	// matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
	// petMatrixVars: ["q" : 22, "s" : 23]
}
```

## 开启矩阵变量

java代码方式：设置UrlPathHelper，removeSemicolonContent=false
xml方式：<mvc:annotation-driven enable-matrix-variables="true"/>

