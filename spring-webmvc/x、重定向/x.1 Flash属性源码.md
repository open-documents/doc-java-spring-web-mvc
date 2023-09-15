



# FlashMap的自动存储

存储FlashMap大致经过以下过程：

## 1）给请求添加类型为FlashMap的属性

请求刚刚进入DispatcherServlet的时候，如果DispatcherServlet的flashMapManager不为空，则创建一个FlashMap对象挂到request的属性上，属性名为"org.springframework.web.servlet.DispatcherServlet.OUTPUT_FLASH_MAP"。源码在org.springframework.web.servlet.DispatcherServlet#doService
```java
public class DispatcherServlet extends FrameworkServlet {
    protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
	    ...
		if (this.flashMapManager != null) {
			FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
			if (inputFlashMap != null) {
				request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
			}
			request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
			request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
		}
	    ...
    }
}
```

## 2）开发者在请求的处理方法上添加RedirectAttributes类型的参数

请求的处理方法上有RedirectAttributes类型的参数时，SpringMVC就用<font color=44cf57>RedirectAttributesMethodArgumentResolver</font>给该方法传一个实际类型为<font color=44cf57>RedirectAttributesModelMap</font>的对象，同时设置ModelAndViewContainer的redirectModel。RedirectAttributesMethodArgumentResolver源码如下：
```java
public class RedirectAttributesMethodArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
	public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
		NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {
		Assert.state(mavContainer != null, "RedirectAttributes argument only supported on regular handler methods");
		ModelMap redirectAttributes;
		if (binderFactory != null) {
			DataBinder dataBinder = binderFactory.createBinder(webRequest, null, DataBinder.DEFAULT_OBJECT_NAME);
			redirectAttributes = new RedirectAttributesModelMap(dataBinder);
		}
		else {
			redirectAttributes  = new RedirectAttributesModelMap();
		}
		mavContainer.setRedirectModel(redirectAttributes);
		return redirectAttributes;
	}
}
```
开发只需要将要保存的属性添加通过RedirectAttributes#addFlashAttribute()添加到参数中即可。

## 3）HandlerAdaptor在执行完请求，将flashAttributes添加到与请求关联的FlashMap对象中

RequstMappingHandlerAdaptor处理完请求方法后，如果model是RedirectAttributes类型就会将flashAttributes加到与该请求关联的FlashMap对象中。关联的FlashMap就是DispatcherServlet在请求刚进来时创建的FlashMap对象。
```java
public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter
		implements BeanFactoryAware, InitializingBean {
    @Nullable
	private ModelAndView getModelAndView(ModelAndViewContainer mavContainer,
			ModelFactory modelFactory, NativeWebRequest webRequest) throws Exception {
		...
        ModelMap model = mavContainer.getModel();
		...
		if (model instanceof RedirectAttributes) {
			Map<String, ?> flashAttributes = ((RedirectAttributes) model).getFlashAttributes();
			HttpServletRequest request = webRequest.getNativeRequest(HttpServletRequest.class);
			if (request != null) {
				RequestContextUtils.getOutputFlashMap(request).putAll(flashAttributes);
			}
		}
		return mav;
	}
}
```
如果请求返回的view是redirect view且ModelAndViewContainer的redirectModel不为空(请求参数上有RedirectAttributes类型的参数，参数解析器就会给redirectModel赋值)，ModelAndViewContainer返回的model就是RedirectAttributes类型的实例。
```java
public class ModelAndViewContainer {
    ... 
    public ModelMap getModel() {
		if (useDefaultModel()) {
			return this.defaultModel;
		}
		else {
			if (this.redirectModel == null) {
				this.redirectModel = new ModelMap();
			}
			return this.redirectModel;
		}
	}
    ...
}
```
## 4)RedirectView在渲染的时候将请求关联的FlashMap对象存储下来

RedirectView渲染时调用org.springframework.web.servlet.support.RequestContextUtils#saveOutputFlashMap，将FlashMap存储下来。
```java
public class RedirectView extends AbstractUrlBasedView implements SmartView {
    
    ... ...
 
    protected void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request,
			HttpServletResponse response) throws IOException {
 
		String targetUrl = createTargetUrl(model, request);
		targetUrl = updateTargetUrl(targetUrl, model, request, response);
 
		// Save flash attributes
		RequestContextUtils.saveOutputFlashMap(targetUrl, request, response);
 
		// Redirect
		sendRedirect(request, response, targetUrl, this.http10Compatible);
	}
    
    ... ...
}

public abstract class RequestContextUtils {
 
    ... ...
 
    public static void saveOutputFlashMap(String location, HttpServletRequest request, HttpServletResponse response) {
		FlashMap flashMap = getOutputFlashMap(request);
		if (CollectionUtils.isEmpty(flashMap)) {
			return;
		}
 
		UriComponents uriComponents = UriComponentsBuilder.fromUriString(location).build();
		flashMap.setTargetRequestPath(uriComponents.getPath());
		flashMap.addTargetRequestParams(uriComponents.getQueryParams());
 
		FlashMapManager manager = getFlashMapManager(request);
		Assert.state(manager != null, "No FlashMapManager. Is this a DispatcherServlet handled request?");
		manager.saveOutputFlashMap(flashMap, request, response);
	}
    
    ... ...
}
```

# FlashMap的自动提取

## 1）DispatcherServlet取出FlashMap对象并关联到request属性

请求刚进来是，如果DispatcherServlet的flashMapManager不为空，则通过flashMapManager提取FlashMap对象，并关联到request的属性上，属性名为"org.springframework.web.servlet.DispatcherServlet.INPUT_FLASH_MAP"。源码在org.springframework.web.servlet.DispatcherServlet#doService()。
## 2）HandlerAdaptor调用请求处理方法前添加到model中

RequstMappingHandlerAdaptor在调用请求的处理方法前，从request属性中取出关联的FlashMap对象，并将其中的键值对初始化到model中。
```java
public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter
		implements BeanFactoryAware, InitializingBean {
    ... ...
	
    @Nullable
	protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {	
    
        ... ...
    
        ModelAndViewContainer mavContainer = new ModelAndViewContainer();
		mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
 
        ... ...
 
    }
}
```
## 3）开发者取属性

开发者只需要在请求的方式上添加Model的参数，就可以取到model里面所有的属性。


# FlashMapManager

FlashMapManager是一个接口，提供了存储和提取FlashMap的接口规范。
```java
public interface FlashMapManager {
 	@Nullable
	FlashMap retrieveAndUpdate(HttpServletRequest request, HttpServletResponse response);
	void saveOutputFlashMap(FlashMap flashMap, HttpServletRequest request, HttpServletResponse response);
}
```
| 方法                 | 描述                                                                                           |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| retrieveAndUpdate()  | 一是找到前面请求保存的且跟当前请求最匹配的FlashMap对象并移除，二移除掉所有已过期的FlashMap对象 |
| saveOutputFlashMap() | 保存FlashMap供后面的请求提取使用                                                               |




web项目有时候需要在一个请求中保存一些属性给接下来的其它请求使用。尤其是重定向场景，也许我们需要给目标请求传递些参数。FlashMap机制就提供了这么一种方式让一个请求传递一些参数给接下来的某个请求。当然要实现这个目的还有别的方式，FlashMap是其中一种，FlashMap机制保证了参数的隐蔽性，不需要将参数传递到前端。

# FlashMap
## 定义
```java
public final class FlashMap extends HashMap<String, Object> implements Comparable<FlashMap> {
	@Nullable
	private String targetRequestPath;
 
	private final MultiValueMap<String, String> targetRequestParams = new LinkedMultiValueMap<>(4);
 
	private long expirationTime = -1;
 
    ... ...
}
```
FlashMap继承了HashMap<String, Object>，所以可以往里面添加各种String-Object的键值对 

## 属性

| 属性                | 描述                                                                       |
| ------------------- | -------------------------------------------------------------------------- |
| targetRequestPath   | 指定了目标请求的路径，即符合该路径的请求才能取到此FlashMap对象             |
| targetRequestParams | 指定了目标请求的参数，即符合该参数的请求才能取到此FlashMap对象             |
| expirationTime      | 指定此FlashMap的过期时间，已经过期的FlashMap对象将移除，不会传递给目标请求 |


FlashMap可以指定绑定条件（targetRequestPath、targetRequestParams），只有匹配条件的请求才能取到该FlashMap对象。如果某个FlashMap没有指定绑定条件，那么紧接着那个请求将得到该FlashMap对象。


# SessionFlashMapManager

FlashMapManager只是接口规范，FlashMap对象保存到哪里、又从哪里提取FlashMap对象。这两点依赖具体的FlashMapManager实现类。
SpringMVC提供了一个实现类org.springframework.web.servlet.support.SessionFlashMapManager。
SessionFlashMapManager就是把FlashMap对象保存到Session中，也是从Session中提取符合条件的FlashMap对象。SessionFlashMapManager将FlashMap对象放在Session属性FLASH_MAPS中。

由于session的局限性，SessionFlashMapManager在单体应用中有效，如果应用是以集群方式部署则由于session在跨机器间无共享，所以将失效。开发者可以实现一个支持集群场景的FlashMapManager。



```java
public class SessionFlashMapManager extends AbstractFlashMapManager {
	private static final String FLASH_MAPS_SESSION_ATTRIBUTE = SessionFlashMapManager.class.getName() + ".FLASH_MAPS";
	// Retrieves saved FlashMap instances from the HTTP session, if any.
	@Override
	@SuppressWarnings("unchecked")
	@Nullable
	protected List<FlashMap> retrieveFlashMaps(HttpServletRequest request) {
		HttpSession session = request.getSession(false);
		return (session != null ? (List<FlashMap>) session.getAttribute(FLASH_MAPS_SESSION_ATTRIBUTE) : null);
	}
	// Saves the given FlashMap instances in the HTTP session.
	@Override
	protected void updateFlashMaps(List<FlashMap> flashMaps, HttpServletRequest request, HttpServletResponse response) {
		WebUtils.setSessionAttribute(request, FLASH_MAPS_SESSION_ATTRIBUTE, (!flashMaps.isEmpty() ? flashMaps : null));
	}
	...
}
```

# 并发问题

参考文档：P918