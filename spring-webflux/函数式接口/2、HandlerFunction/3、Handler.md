可以将Handler定义为一个lamda表达式。
```java
HandlerFunction<ServerResponse> helloWorld = request -> ServerResponse.ok().body("Hello World");
```

通过lamda表达式定义Handler是方便的，但是通常存在多个Handler，这是不容易管理的。因此可以将多个相关的Handler定义在一起，表现为@Controller。
```java
import static org.springframework.http.MediaType.APPLICATION_JSON; 
import static org.springframework.web.reactive.function.server.ServerResponse.ok; 
public class PersonHandler {   
	private final PersonRepository repository;   
	public PersonHandler(PersonRepository repository) {   
		this.repository = repository; 
	}
	public ServerResponse listPeople(ServerRequest request) {
		List<Person> people = repository.allPeople(); 
		return ok().contentType(APPLICATION_JSON).body(people);
	} 
	public ServerResponse createPerson(ServerRequest request) throws Exception {
		Person person = request.body(Person.class);
		repository.savePerson(person); 
		return ok().build();
	} 
	public ServerResponse getPerson(ServerRequest request) { 
		int personId = Integer.parseInt(request.pathVariable("id")); 
		Person person = repository.getPerson(personId); 
		if (person != null) {
			return ok().contentType(APPLICATION_JSON).body(person);
		} 
		else {
			return ServerResponse.notFound().build();
		}
	}
}
```

# 对ServerRequest进行验证

可以使用Spring验证机制对请求体进行验证。
```java
public class PersonHandler { 
	private final Validator validator = new PersonValidator();
	public ServerResponse createPerson(ServerRequest request) { 
		Person person = request.body(Person.class); 
		validate(person);
		repository.savePerson(person); 
		return ok().build();
	}
	private void validate(Person person) { 
		Errors errors = new BeanPropertyBindingResult(person, "person");
		validator.validate(person, errors);
		if (errors.hasErrors()) { 
			throw new ServerWebInputException(errors.toString()); 
		}
	}
}
```

