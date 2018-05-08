---
layout: post
category : Tech
title : Spring MVC 自定义接口参数解析器
tags : [java]
excerpt: 在开发中，MVC提供的接口参数解析类型满足不了需求，这时就需要去自定义自己的参数解析器，无论是XML，JSON等数据类型。
---
{% include JB/setup %}

在开发中，MVC提供的接口参数解析类型满足不了需求，这时就需要去自定义自己的参数解析器，无论是XML，JSON等数据类型。其实Spring MVC已经提供了方法来解决该问题。可使用`mvc:argument-resolvers`来注入自定义的参数解析器。

```xml
<mvc:argument-resolvers>
    <bean />
</mvc:argument-resolvers>
```

> Configures HandlerMethodArgumentResolver types to support custom controller method argument types. Using this option does not override the built-in support for resolving handler method arguments. To customize the built-in support for argument resolution configure RequestMappingHandlerAdapter directly.

通过以上描述，我们知道可通过该配置添加Controller的参数解析器。

Spring MVC 3.2以前通过`AnnotationMethodHandlerAdapter`类来实现，3.2以后被弃用，改为`RequestMappingHandlerAdapter`类，二者都实现了`HandlerAdapter`的`handle`方法。

```java
// AnnotationMethodHandlerAdapter
public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
	Class<?> clazz = ClassUtils.getUserClass(handler);
	Boolean annotatedWithSessionAttributes = this.sessionAnnotatedClassesCache.get(clazz);
	.....
	// 调用invokeHandlerMethod
	if (this.synchronizeOnSession) {
		HttpSession session = request.getSession(false);
		if (session != null) {
			Object mutex = WebUtils.getSessionMutex(session);
			synchronized (mutex) {
				return invokeHandlerMethod(request, response, handler);
			}
		}
	}

	return invokeHandlerMethod(request, response, handler);
}

protected ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, Object handler)
		throws Exception {

	ServletHandlerMethodResolver methodResolver = getMethodResolver(handler);
  	// 调用ServletHandlerMethodResolver的resolveHandlerMethod方法，该方法负责解析参数
	Method handlerMethod = methodResolver.resolveHandlerMethod(request);
	......
	return mav;
}



// RequestMappingHandlerAdapter
// RequestMappingHandlerAdapter继承于AbstractHandlerMethodAdapter，AbstractHandlerMethodAdapter是个抽象类，
// 该类实现了HandlerAdapter的handle方法，该方法调用handleInternal抽象方法，并在handleInternal中进行参数解析设置
protected ModelAndView handleInternal(HttpServletRequest request,
        HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

	......

    // Execute invokeHandlerMethod in synchronized block if required.
    if (this.synchronizeOnSession) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            Object mutex = WebUtils.getSessionMutex(session);
            synchronized (mutex) {
                mav = invokeHandlerMethod(request, response, handlerMethod);
            }
        }
        else {
            // No HttpSession available -> no mutex necessary
            mav = invokeHandlerMethod(request, response, handlerMethod);
        }
    }
    else {
        // No synchronization on session demanded at all...
        mav = invokeHandlerMethod(request, response, handlerMethod);
    }

    ......

    return mav;
}

// 由该方法进行参数解析器配置
protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

	ServletWebRequest webRequest = new ServletWebRequest(request, response);
	try {
		WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
		ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

		ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
		// 把this.argumentResolvers注入到ServletInvocableHandlerMethod中
		invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
		invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);

		......

        if (asyncManager.hasConcurrentResult()) {
            Object result = asyncManager.getConcurrentResult();
            mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
            asyncManager.clearConcurrentResult();
            if (logger.isDebugEnabled()) {
                logger.debug("Found concurrent result value [" + result + "]");
            }
            invocableMethod = invocableMethod.wrapConcurrentResult(result);
        }

        // 请求参数和返回值处理
        invocableMethod.invokeAndHandle(webRequest, mavContainer);
        if (asyncManager.isConcurrentHandlingStarted()) {
            return null;
        }

		return getModelAndView(mavContainer, modelFactory, webRequest);
	}
	finally {
		webRequest.requestCompleted();
	}
}

/**
 * Return the list of argument resolvers to use including built-in resolvers
 * and custom resolvers provided via {@link #setCustomArgumentResolvers}.
 */
 // 该方法会在afterPropertiesSet中调用，也就是说在属性设置完成后会创建参数解析器
private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
	List<HandlerMethodArgumentResolver> resolvers = new ArrayList<HandlerMethodArgumentResolver>();

	// Annotation-based argument resolution
	resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), false));
	resolvers.add(new RequestParamMapMethodArgumentResolver());
	resolvers.add(new PathVariableMethodArgumentResolver());
	resolvers.add(new PathVariableMapMethodArgumentResolver());
	resolvers.add(new MatrixVariableMethodArgumentResolver());
	resolvers.add(new MatrixVariableMapMethodArgumentResolver());
 	// GET请求会使用ServletModelAttributeMethodProcessor进行参数解析
	resolvers.add(new ServletModelAttributeMethodProcessor(false));
	// 带有RequestBody注解的请求由RequestResponseBodyMethodProcessor处理
	resolvers.add(new RequestResponseBodyMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
	resolvers.add(new RequestPartMethodArgumentResolver(getMessageConverters(), this.requestResponseBodyAdvice));
	resolvers.add(new RequestHeaderMethodArgumentResolver(getBeanFactory()));
	resolvers.add(new RequestHeaderMapMethodArgumentResolver());
	resolvers.add(new ServletCookieValueMethodArgumentResolver(getBeanFactory()));
	resolvers.add(new ExpressionValueMethodArgumentResolver(getBeanFactory()));
	resolvers.add(new SessionAttributeMethodArgumentResolver());
	resolvers.add(new RequestAttributeMethodArgumentResolver());

	// Type-based argument resolution
	resolvers.add(new ServletRequestMethodArgumentResolver());
	resolvers.add(new ServletResponseMethodArgumentResolver());
	resolvers.add(new HttpEntityMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
	resolvers.add(new RedirectAttributesMethodArgumentResolver());
	resolvers.add(new ModelMethodProcessor());
	resolvers.add(new MapMethodProcessor());
	resolvers.add(new ErrorsMethodArgumentResolver());
	resolvers.add(new SessionStatusMethodArgumentResolver());
	resolvers.add(new UriComponentsBuilderMethodArgumentResolver());

	// Custom arguments
	// 自定义的解析器会在此处被添加
	if (getCustomArgumentResolvers() != null) {
		resolvers.addAll(getCustomArgumentResolvers());
	}

	// Catch-all
	resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));
	resolvers.add(new ServletModelAttributeMethodProcessor(true));

	return resolvers;
}



// ServletInvocableHandlerMethod
public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
        Object... providedArgs) throws Exception {
    // 处理请求参数, 调用父类InvocableHandlerMethod的invokeForRequest方法
    Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
    setResponseStatus(webRequest);

    ......
}

// InvocableHandlerMethod
public Object invokeForRequest(NativeWebRequest request, ModelAndViewContainer mavContainer,
        Object... providedArgs) throws Exception {
    // 进行参数解析处理
    Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);

  	......

    return returnValue;
}

/**
 * Get the method argument values for the current request.
 */
// 真正的参数解析在此处完成
private Object[] getMethodArgumentValues(NativeWebRequest request, ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {
    MethodParameter[] parameters = getMethodParameters();
    Object[] args = new Object[parameters.length];
    for (int i = 0; i < parameters.length; i++) {
        MethodParameter parameter = parameters[i];
        parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
        args[i] = resolveProvidedArgument(parameter, providedArgs);
        if (args[i] != null) {
            continue;
        }
        if (this.argumentResolvers.supportsParameter(parameter)) {
            try {
            	// 参数解析
              	// 调用HandlerMethodArgumentResolver的实现类来进行处理
                args[i] = this.argumentResolvers.resolveArgument(
                        parameter, mavContainer, request, this.dataBinderFactory);
                continue;
            }
            catch (Exception ex) {
                if (logger.isDebugEnabled()) {
                    logger.debug(getArgumentResolutionErrorMessage("Failed to resolve", i), ex);
                }
                throw ex;
            }
        }
        if (args[i] == null) {
            throw new IllegalStateException("Could not resolve method parameter at index " +
                    parameter.getParameterIndex() + " in " + parameter.getMethod().toGenericString() +
                    ": " + getArgumentResolutionErrorMessage("No suitable resolver for", i));
        }
    }
    return args;
}
```

以上就是参数解析的大概过程，如下图所示：

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentScriptType="application/ecmascript" contentStyleType="text/css" height="274px" preserveAspectRatio="none" style="width:1752px;height:274px;" version="1.1" viewBox="0 0 1752 274" width="1752px" zoomAndPan="magnify"><defs><filter height="300%" id="f1qtmtzpfsdrhs" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="76" x2="76" y1="38.4883" y2="234.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="218" x2="218" y1="38.4883" y2="234.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="410" x2="410" y1="38.4883" y2="234.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="659" x2="659" y1="38.4883" y2="234.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="884" x2="884" y1="38.4883" y2="234.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="1321" x2="1321" y1="38.4883" y2="234.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="1619" x2="1619" y1="38.4883" y2="234.3516"></line><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="133" x="8" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="119" x="15" y="23.5352">DispatcherServlet</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="133" x="8" y="233.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="119" x="15" y="253.8867">DispatcherServlet</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="123" x="155" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="109" x="162" y="23.5352">HandlerAdapter</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="123" x="155" y="233.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="109" x="162" y="253.8867">HandlerAdapter</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="232" x="292" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="218" x="299" y="23.5352">AbstractHandlerMethodAdapter</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="232" x="292" y="233.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="218" x="299" y="253.8867">AbstractHandlerMethodAdapter</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="238" x="538" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="224" x="545" y="23.5352">RequestMappingHandlerAdapter</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="238" x="538" y="233.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="224" x="545" y="253.8867">RequestMappingHandlerAdapter</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="185" x="790" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="171" x="797" y="23.5352">InvocableHandlerMethod</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="185" x="790" y="233.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="171" x="797" y="253.8867">InvocableHandlerMethod</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="321" x="1159" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="307" x="1166" y="23.5352">HandlerMethodArgumentResolverComposite</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="321" x="1159" y="233.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="307" x="1166" y="253.8867">HandlerMethodArgumentResolverComposite</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="247" x="1494" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="233" x="1501" y="23.5352">HandlerMethodArgumentResolver</text><rect fill="#FEFECE" filter="url(#f1qtmtzpfsdrhs)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="247" x="1494" y="233.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="233" x="1501" y="253.8867">HandlerMethodArgumentResolver</text><polygon fill="#A80036" points="206.5,65.4883,216.5,69.4883,206.5,73.4883,210.5,69.4883" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="76.5" x2="212.5" y1="69.4883" y2="69.4883"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="72" x="83.5" y="65.0566">doDispatch</text><polygon fill="#A80036" points="398,94.7988,408,98.7988,398,102.7988,402,98.7988" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="218.5" x2="404" y1="98.7988" y2="98.7988"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="42" x="225.5" y="94.3672">handle</text><polygon fill="#A80036" points="647,124.1094,657,128.1094,647,132.1094,651,128.1094" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="410" x2="653" y1="128.1094" y2="128.1094"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="90" x="417" y="123.6777">handleInternal</text><polygon fill="#A80036" points="872.5,153.4199,882.5,157.4199,872.5,161.4199,876.5,157.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="659" x2="878.5" y1="157.4199" y2="157.4199"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="138" x="666" y="152.9883">invokeHandlerMethod</text><polygon fill="#A80036" points="1309.5,182.7305,1319.5,186.7305,1309.5,190.7305,1313.5,186.7305" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="884.5" x2="1315.5" y1="186.7305" y2="186.7305"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="413" x="891.5" y="182.2988">invokeAndHandle,  invokeForRequest, getMethodArgumentValues</text><polygon fill="#A80036" points="1607.5,212.041,1617.5,216.041,1607.5,220.041,1611.5,216.041" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="1321.5" x2="1613.5" y1="216.041" y2="216.041"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="233" x="1328.5" y="211.6094">supportsParameter, resolveArgument</text></g></svg>

下面我们来看看如何定义自己的参数解析器，当请求为GET时，使用Spring默认的GET请求的参数解析器，请求为POST时，使用messageConvert解析器，与使用@RequestBody效果一致， 本例使用Jackson作为JSON解析器。

**spring-mvc.xml**

```xml
<mvc:annotation-driven>
	<mvc:argument-resolvers>
		<bean id="multiArgumentResolvers" class="net.cofcool.annotation.MultiArgumentResolvers" />
	</mvc:argument-resolvers>
</mvc:annotation-driven>
```

**MultiRequestTypes**

```java
package net.cofcool.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MultiRequestTypes {

}
```

**MultiArgumentResolvers**

```java
package net.cofcool.annotation;

import com.fasterxml.jackson.annotation.JsonInclude.Include;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import javax.servlet.http.HttpServletRequest;
import lombok.extern.slf4j.Slf4j;
import org.springframework.core.MethodParameter;
import org.springframework.http.HttpMethod;
import org.springframework.http.MediaType;
import org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
import org.springframework.http.server.ServletServerHttpRequest;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;
import org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor;
import org.springframework.web.servlet.mvc.method.annotation.ServletModelAttributeMethodProcessor;


@Slf4j
public class MultiArgumentResolvers implements HandlerMethodArgumentResolver {

    // POST请求，Content-Type为JSON
    private HandlerMethodArgumentResolver jsonResolver;

    // get请求
    private HandlerMethodArgumentResolver requestParamResolver = new ServletModelAttributeMethodProcessor(false);


    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return hasMultiRequestTypesAnnotation(parameter);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        HttpServletRequest request = webRequest.getNativeRequest(HttpServletRequest.class);

        if (checkGetRequest(parameter, request)) {
            return requestParamResolver
                .resolveArgument(parameter, mavContainer, webRequest, binderFactory);
        }

        try {
            return getJsonResolver().resolveArgument(parameter, mavContainer, webRequest, binderFactory);
        } catch (NullPointerException e) {
            log.info("resolve json error, try parse by get processor: " , e);
            return requestParamResolver
                .resolveArgument(parameter, mavContainer, webRequest, binderFactory);
        }
    }

    private boolean checkGetRequest(MethodParameter parameter, HttpServletRequest request) {
        return
            HttpMethod.GET.matches(request.getMethod()) && hasMultiRequestTypesAnnotation(parameter);
    }

    private boolean hasMultiRequestTypesAnnotation(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(MultiRequestTypes.class);
    }

    @SuppressWarnings("unchecked")
    private HandlerMethodArgumentResolver getJsonResolver() {
        if (this.jsonResolver == null) {
            List resolvers = new ArrayList<>();
            resolvers.add(getJackson2HttpMessageConverter());

            jsonResolver = new RequestResponseBodyMethodProcessor(resolvers);
        }

        return jsonResolver;
    }

    AbstractJackson2HttpMessageConverter getJackson2HttpMessageConverter() {
        AbstractJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();

        List<MediaType> mediaTypes = new ArrayList<>();
        mediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
        converter.setSupportedMediaTypes(mediaTypes);
        converter.setPrettyPrint(true);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setSerializationInclusion(Include.NON_NULL);

        return converter;
    }
}
```
