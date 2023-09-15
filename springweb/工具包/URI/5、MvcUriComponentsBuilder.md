	参考文档:P967

Springmvc提供一种借用控制器及控制器方法创建URI的方式。

```java
@Controller 
@RequestMapping("/hotels/{hotel}") 
public class BookingController {   
	@GetMapping("/bookings/{booking}")
	public ModelAndView getBooking(@PathVariable Long booking) {   
		// ...
	}
}
```

```java
UriComponentsBuilder builder = MvcUriComponentsBuilder.fromMethodName(BookingController.class, "getBooking", 21);

UriComponentsBuilder builder = MvcUriComponentsBuilder.fromMethodCall(on(BookingController.class).getBooking(21))

```