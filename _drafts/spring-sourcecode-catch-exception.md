

AbstractHandlerExceptionResolver

HandlerExceptionResolver
```java
public interface HandlerExceptionResolver {

	@Nullable
	ModelAndView resolveException(
			HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex);

}
```