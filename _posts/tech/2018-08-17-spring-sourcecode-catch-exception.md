---
layout: post
category: Tech
title: Spring MVC 异常处理源码解析及自定义异常解析器
tags: [java, spring, sourcode]
excerpt: Spring MVC如何通过HandlerExceptionResolver来处理异常？
---

{% include JB/setup %}

开发中，异常的处理至关重要，例如对用户来说友好的提示，无扰的报错；对开发者来说便捷的异常捕捉，完善的日志，代码的简化，开发效率的提高等。

Spring MVC定义了`HandlerExceptionResolver`接口，实现该接口的类负责解析应用中的各种异常异常，包括`checked exception`, `runtime exception`等。

```java
public interface HandlerExceptionResolver {

  ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex);

}
```

目录：

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 异常解析器初始化](#1-异常解析器初始化)
* [2. 异常解析](#2-异常解析)
	* [2.1 DispatcherServlet处理流程](#21-dispatcherservlet处理流程)
	* [2.2 HandlerExceptionResolver处理流程](#22-handlerexceptionresolver处理流程)
		* [2.2.1 ExceptionHandlerExceptionResolver](#221-exceptionhandlerexceptionresolver)
		* [2.2.2 ResponseStatusExceptionResolver](#222-responsestatusexceptionresolver)
		* [2.2.3 DefaultHandlerExceptionResolver](#223-defaulthandlerexceptionresolver)
		* [2.2.4 HandlerExceptionResolverComposite](#224-handlerexceptionresolvercomposite)
	* [2.3 使用@EnableWebMvc注解](#23-使用enablewebmvc注解)
	* [2.4 自定义异常解析器](#24-自定义异常解析器)

<!-- /code_chunk_output -->


## 1. 异常解析器初始化

Spring MVC的`dispatcher`默认配置的异常处理器：

* org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver
* org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver
* org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

我们来看看`DispatcherServlet`异常处理的相关代码（省略与异常处理无关的代码）。

**DispatcherServlet**：

初始化异常处理器，默认`this.detectAllHandlerExceptionResolvers`为`true`，也就是说优先寻找实现`HandlerExceptionResolvers`的类，若未发现则创建默认解析器。

```java
private void initHandlerExceptionResolvers(ApplicationContext context) {
  this.handlerExceptionResolvers = null;

  if (this.detectAllHandlerExceptionResolvers) {
    // 在上下文中寻找 HandlerExceptionResolvers，包含父容器的上下文
    Map<String, HandlerExceptionResolver> matchingBeans = BeanFactoryUtils
    .beansOfTypeIncludingAncestors(context, HandlerExceptionResolver.class, true, false);
    if (!matchingBeans.isEmpty()) {
      this.handlerExceptionResolvers = new ArrayList<>(matchingBeans.values());
      // 根据sort排序
      AnnotationAwareOrderComparator.sort(this.handlerExceptionResolvers);
    }
  }
  else {
    try {
      // 在上下文中寻找名称为HANDLER_EXCEPTION_RESOLVER_BEAN_NAME的类，也就是handlerExceptionResolver
      HandlerExceptionResolver her =
      context.getBean(HANDLER_EXCEPTION_RESOLVER_BEAN_NAME, HandlerExceptionResolver.class);
      this.handlerExceptionResolvers = Collections.singletonList(her);
    }
    catch (NoSuchBeanDefinitionException ex) {
    }
  }

  // 确保this.handlerExceptionResolvers不为null
  // 调用getDefaultStrategies函数从默认配置文件中读取
  if (this.handlerExceptionResolvers == null) {
    this.handlerExceptionResolvers = getDefaultStrategies(context, HandlerExceptionResolver.class);
    if (logger.isDebugEnabled()) {
      logger.debug("No HandlerExceptionResolvers found in servlet '" + getServletName() + "': using default");
    }
  }
}
```

默认配置，根据传入的`strategyInterface`创建对应的实例，
```java
protected <T> List<T> getDefaultStrategies(ApplicationContext context, Class<T> strategyInterface) {
  String key = strategyInterface.getName();
  // 读取 strategyInterface对应的默认配置
  String value = defaultStrategies.getProperty(key);
  if (value != null) {
    // 根据，分割字符串，读取配置的类名
    String[] classNames = StringUtils.commaDelimitedListToStringArray(value);
    List<T> strategies = new ArrayList<>(classNames.length);
    for (String className : classNames) {
      try {
        // 根据类名创建实例，并添加到数组中
        Class<?> clazz = ClassUtils.forName(className, DispatcherServlet.class.getClassLoader());
        Object strategy = createDefaultStrategy(context, clazz);
        strategies.add((T) strategy);
      }
      catch (ClassNotFoundException ex) {
        throw new BeanInitializationException(
        "Could not find DispatcherServlet's default strategy class [" + className +
        "] for interface [" + key + "]", ex);
      }
      catch (LinkageError err) {
          throw new BeanInitializationException(
            "Unresolvable class definition for DispatcherServlet's default strategy class [" +
            className + "] for interface [" + key + "]", err);
      }
    }
    return strategies;
  }
  else {
    return new LinkedList<>();
  }
}
```

从`DispatcherServlet.properties`读取默认配置。
```java
private static final String DEFAULT_STRATEGIES_PATH = "DispatcherServlet.properties";

private static final Properties defaultStrategies;

static {
  // 载入Spring MVC的默认配置
  try {
    ClassPathResource resource = new ClassPathResource(DEFAULT_STRATEGIES_PATH, DispatcherServlet.class);
    defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
  }
  catch (IOException ex) {
    throw new IllegalStateException("Could not load '" + DEFAULT_STRATEGIES_PATH + "': " + ex.getMessage());
  }
}
```

`DispatcherServlet.properties`中定义的`HandlerExceptionResolvers`内容如下:

```properties
org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,\
  org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
  org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

```

## 2. 异常解析

### 2.1 DispatcherServlet处理流程

调用接口时，DispatcherServlet通过`doDispatch`方法处理一系列流程，当业务逻辑处理完成返回时，调用`processDispatchResult`来进行页面渲染，异常处理等。当异常不是`ModelAndViewDefiningException`时，调用`processHandlerException`来处理异常。

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
  @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
  @Nullable Exception exception) throws Exception {

  boolean errorView = false;

  // 处理异常
  if (exception != null) {
    if (exception instanceof ModelAndViewDefiningException) {
      logger.debug("ModelAndViewDefiningException encountered", exception);
      mv = ((ModelAndViewDefiningException) exception).getModelAndView();
    }
    // 处理非ModelAndViewDefiningException异常
    else {
      Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
      mv = processHandlerException(request, response, handler, exception);
      errorView = (mv != null);
    }
  }

  ...
}
```

调用`processHandlerException`处理异常，通过异常处理器解析异常，返回对应的结果。

```java
protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response,
  @Nullable Object handler, Exception ex) throws Exception {

  // 遍历HandlerExceptionResolvers中的元素
  // 如果exMv不为null，则说明异常处理结束，跳出循环
  ModelAndView exMv = null;
  if (this.handlerExceptionResolvers != null) {
    for (HandlerExceptionResolver handlerExceptionResolver : this.handlerExceptionResolvers) {
      exMv = handlerExceptionResolver.resolveException(request, response, handler, ex);
      if (exMv != null) {
        break;
      }
    }
  }
  if (exMv != null) {
    if (exMv.isEmpty()) {
      request.setAttribute(EXCEPTION_ATTRIBUTE, ex);
      return null;
    }
    // 配置异常对应的view，若无则添加默认view
    if (!exMv.hasView()) {
      String defaultViewName = getDefaultViewName(request);
      if (defaultViewName != null) {
        exMv.setViewName(defaultViewName);
      }
    }
    if (logger.isDebugEnabled()) {
      logger.debug("Handler execution resulted in exception - forwarding to resolved error view: " + exMv, ex);
    }
    // 暴露异常，也就是让requset包含javax.servlet.error.status_code等数据
    WebUtils.exposeErrorRequestAttributes(request, ex, getServletName());
    return exMv;
  }

  throw ex;
}
```

以上就是DispatcherServlet处理异常的过程，下面我们来看看HandlerExceptionResolvers是如何处理的。

### 2.2 HandlerExceptionResolver处理流程

我们来逐个解析Spring MVC提供的异常解析器。

#### 2.2.1 ExceptionHandlerExceptionResolver

该类默认优先级较高,负责解析含有`@ControllerAdvice`和`@ExceptionHandler`注解的Controller抛出的异常。

```java
protected ModelAndView doResolveHandlerMethodException(HttpServletRequest request,
  HttpServletResponse response, @Nullable HandlerMethod handlerMethod, Exception exception) {
  ServletInvocableHandlerMethod exceptionHandlerMethod = getExceptionHandlerMethod(handlerMethod, exception);
  if (exceptionHandlerMethod == null) {
    return null;
  }
  ...

  try {
    if (logger.isDebugEnabled()) {
      logger.debug("Invoking @ExceptionHandler method: " + exceptionHandlerMethod);
    }
    // 通过ServletInvocableHandlerMethod处理异常
    Throwable cause = exception.getCause();
    if (cause != null) {
      // 处理该异常包装的异常
      exceptionHandlerMethod.invokeAndHandle(webRequest, mavContainer, exception, cause, handlerMethod);
    }
    else {
      // 仅处理异常
      exceptionHandlerMethod.invokeAndHandle(webRequest, mavContainer, exception, handlerMethod);
    }
  }
  catch (Throwable invocationEx) {
    ....
    return null;
  }

  ...
}

// 构建ServletInvocableHandlerMethod实例
protected ServletInvocableHandlerMethod getExceptionHandlerMethod(
  @Nullable HandlerMethod handlerMethod, Exception exception) {

  Class<?> handlerType = null;

  if (handlerMethod != null) {
    // 创建并缓存Controller中定义的异常
    handlerType = handlerMethod.getBeanType();
    ExceptionHandlerMethodResolver resolver = this.exceptionHandlerCache.get(handlerType);
    if (resolver == null) {
      // 创建resolver，并缓存
      resolver = new ExceptionHandlerMethodResolver(handlerType);
      this.exceptionHandlerCache.put(handlerType, resolver);
    }
    Method method = resolver.resolveMethod(exception);
    if (method != null) {
      // 如果在Controller中定义了ExceptionHandler注解标注的方法
      // 则创建 ServletInvocableHandlerMethod
      return new ServletInvocableHandlerMethod(handlerMethod.getBean(), method);
    }
    // 检测是否使用AOP代理，如有AOP，则把controller替换为代理类
    if (Proxy.isProxyClass(handlerType)) {
      handlerType = AopUtils.getTargetClass(handlerMethod.getBean());
    }
  }

  // 根据ControllerAdvice的定义来创建 ServletInvocableHandlerMethod
  // 项目启动时根据扫描结果，通过afterPropertiesSet方法创建this.exceptionHandlerAdviceCache
  for (Map.Entry<ControllerAdviceBean, ExceptionHandlerMethodResolver> entry : this.exceptionHandlerAdviceCache.entrySet()) {
    ControllerAdviceBean advice = entry.getKey();
    // controller使用了ControllerAdvice注解
    if (advice.isApplicableToBeanType(handlerType)) {
      ExceptionHandlerMethodResolver resolver = entry.getValue();
      Method method = resolver.resolveMethod(exception);
      if (method != null) {
        return new ServletInvocableHandlerMethod(advice.resolveBean(), method);
      }
    }
  }

  return null;
}
```

#### 2.2.2 ResponseStatusExceptionResolver

负责解析返回状态码和带有`@ResponseStatus`的Controller。

```java
protected ModelAndView doResolveException(
  HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {

  try {
    if (ex instanceof ResponseStatusException) {
      return resolveResponseStatusException((ResponseStatusException) ex, request, response, handler);
    }

    // 解析 ResponseStatus
    ResponseStatus status = AnnotatedElementUtils.findMergedAnnotation(ex.getClass(), ResponseStatus.class);
    if (status != null) {
      return resolveResponseStatus(status, request, response, handler, ex);
    }

    // 解析其它异常
    if (ex.getCause() instanceof Exception) {
      ex = (Exception) ex.getCause();
      return doResolveException(request, response, handler, ex);
    }
  }
  catch (Exception resolveEx) {
    logger.warn("ResponseStatus handling resulted in exception", resolveEx);
  }
  return null;
}
````

#### 2.2.3 DefaultHandlerExceptionResolver

转换异常为4xx，55xx等Http协议规范错误码。

```java
protected ModelAndView doResolveException(
  HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {

  try {
    // 405
    if (ex instanceof HttpRequestMethodNotSupportedException) {
      return handleHttpRequestMethodNotSupported(
        (HttpRequestMethodNotSupportedException) ex, request, response, handler);
    }
    // 415
    else if (ex instanceof HttpMediaTypeNotSupportedException) {
      return handleHttpMediaTypeNotSupported(
        (HttpMediaTypeNotSupportedException) ex, request, response, handler);
    }
    // 406
    else if (ex instanceof HttpMediaTypeNotAcceptableException) {
      return handleHttpMediaTypeNotAcceptable(
        (HttpMediaTypeNotAcceptableException) ex, request, response, handler);
    }
    // 500
    else if (ex instanceof MissingPathVariableException) {
      return handleMissingPathVariable(
        (MissingPathVariableException) ex, request, response, handler);
    }
  ......
  }
  catch (Exception handlerException) {
    if (logger.isWarnEnabled()) {
      logger.warn("Handling of [" + ex.getClass().getName() + "] resulted in exception", handlerException);
    }
  }
  return null;
}
```

#### 2.2.4 HandlerExceptionResolverComposite

HandlerExceptionResolverComposite一个特殊的异常解析器，它自身不具备异常解析功能，而是通过它的`resolvers`属性来解析异常。如果使用@EnableWebMvc注解，则会创建该异常解析器。

```java
// 存储HandlerExceptionResolver
private List<HandlerExceptionResolver> resolvers;

public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response,
  @Nullable Object handler,Exception ex) {

  // 遍历 resolvers，如mav不为null则跳出循环
  if (this.resolvers != null) {
    for (HandlerExceptionResolver handlerExceptionResolver : this.resolvers) {
      ModelAndView mav = handlerExceptionResolver.resolveException(request, response, handler, ex);
      if (mav != null) {
        return mav;
      }
    }
  }
  return null;
}
```

### 2.3 使用@EnableWebMvc注解

在开发中，通过java类的方式来配置Spring，如使用注解:
```java
@Configuration
@EnableWebMvc
```

它可以创建一系列默认对象，如`mvcValidator`,`handlerExceptionResolver`, `requestMappingHandlerAdapter`, `requestMappingHandlerMapping`等。具体的创建配置工作由`WebMvcConfigurationSupport`完成。

我们来看看**WebMvcConfigurationSupport**是如何配置异常解析器的。

```java
@Bean
public HandlerExceptionResolver handlerExceptionResolver() {
  List<HandlerExceptionResolver> exceptionResolvers = new ArrayList<>();
  configureHandlerExceptionResolvers(exceptionResolvers);
  if (exceptionResolvers.isEmpty()) {
    // 添加默认异常解析器
    addDefaultHandlerExceptionResolvers(exceptionResolvers);
  }
  // 抽象方法，子类可添加自定义 HandlerExceptionResolver
  extendHandlerExceptionResolvers(exceptionResolvers);

  // 创建 HandlerExceptionResolverComposite
  HandlerExceptionResolverComposite composite = new HandlerExceptionResolverComposite();
  // 优先级为0，较高
  composite.setOrder(0);
  // 为 composite 设置默认异常解析器
  composite.setExceptionResolvers(exceptionResolvers);
  return composite;
}

// 添加默认异常解析器
protected final void addDefaultHandlerExceptionResolvers(List<HandlerExceptionResolver> exceptionResolvers) {
  ExceptionHandlerExceptionResolver exceptionHandlerResolver = createExceptionHandlerExceptionResolver();

  ...

  exceptionResolvers.add(exceptionHandlerResolver);

  ResponseStatusExceptionResolver responseStatusResolver = new ResponseStatusExceptionResolver();

  ...

  exceptionResolvers.add(responseStatusResolver);

  exceptionResolvers.add(new DefaultHandlerExceptionResolver());
}
```

### 2.4 自定义异常解析器

通过以上分析，我们对Spring MVC的异常处理流程有了较为清晰的认识。如果要自定义异常解析器，需实现`HandlerExceptionResolver`接口，Spring MVC中的`AbstractHandlerExceptionResolver`实现了该接口，我们继承该类实现自定义逻辑即可。要启用该解析器，可使用上文提到的注解并实现`WebMvcConfigurer`接口的`extendHandlerExceptionResolvers`方法，也可直接在自定义类上使用`@Component`注解。

示例代码:

```java
/**
 * @author CofCool
 * @date 2018/8/17.
 */

@Component
public class ExampleHandlerExceptionResolver extends AbstractHandlerExceptionResolver {

  @Override
  protected ModelAndView doResolveException(HttpServletRequest request,
      HttpServletResponse response, Object handler, Exception ex) {

      if (this.logger.isErrorEnabled()) {
        OutputStream outputStream = new ByteArrayOutputStream();
        ex.printStackTrace(new PrintStream(outputStream));

        this.logger.error(outputStream.toString());
      }

    return null;
  }
}
```
