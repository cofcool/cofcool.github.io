---
layout: post
category : Tech
title : Spring Web 异常处理处理与 ProblemDetail
tags : [java, spring]
excerpt: Web 接口请求出现错误在所难免，那如何能更好的描述请求错误信息呢？为此，IETF 制定了……
---
{% include JB/setup %}

Web 接口请求出现错误在所难免，那如何能更好的描述请求错误信息呢？为此，IETF 制定了 [RFC 7807 - Problem Details for HTTP APIs](https://datatracker.ietf.org/doc/html/rfc7807)，它规范了接口请求错误时的如何响应，支持 JSON 和 XML。例如，请求 403 错误时的 JSON 响应结构：

```
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json
Content-Language: en

{
  "type": "https://example.com/probs/out-of-credit",
  "title": "You do not have enough credit.",
  "detail": "Your current balance is 30, but that costs 50.",
  "instance": "/account/12345/msgs/abc",
  "balance": 30,
  "accounts": ["/account/12345", "/account/67890"]
}
```

* **type**， 描述问题的 URL，例如错误帮助页面
* **title**，简单明了的描述发生的问题
* **status**，HTTP 状态码
* **detail**，更详细的问题描述
* **instance**，请求的 URL
* *balance，accounts*，自定义字段

Spring 作为业内主流框架，当然也会实现该规范。

Spring Web 6 添加 `org.springframework.http.ProblemDetail`，配合 `ExceptionHandlerExceptionResolver` 把异常信息转换为“RFC 7807”规定的格式，下面我们来看看具体实现细节。

开发中异常处理常用的方式为使用 `@ControllerAdvice` 和 `@ExceptionHandler` 注解拦截异常并转换为易读的错误描述。ProblemDetail 也是使用类似的方式处理的，实现参考 `ProblemDetailsExceptionHandler` 类。

```java
// ProblemDetailsExceptionHandler
@ControllerAdvice
class ProblemDetailsExceptionHandler extends ResponseEntityExceptionHandler {

}

// ResponseEntityExceptionHandler
public abstract class ResponseEntityExceptionHandler implements MessageSourceAware {

  @ExceptionHandler({
    HttpRequestMethodNotSupportedException.class,
    HttpMediaTypeNotSupportedException.class,
    HttpMediaTypeNotAcceptableException.class,
    MissingPathVariableException.class,
    MissingServletRequestParameterException.class,
    MissingServletRequestPartException.class,
    ServletRequestBindingException.class,
    MethodArgumentNotValidException.class,
    NoHandlerFoundException.class,
    AsyncRequestTimeoutException.class,
    ErrorResponseException.class,
    ConversionNotSupportedException.class,
    TypeMismatchException.class,
    HttpMessageNotReadableException.class,
    HttpMessageNotWritableException.class,
    BindException.class
    })
  @Nullable
  public final ResponseEntity<Object> handleException(Exception ex, WebRequest request) throws Exception {
    ...
    else if (ex instanceof TypeMismatchException theEx) {
      return handleTypeMismatch(theEx, headers, HttpStatus.BAD_REQUEST, request);
    }
    else if (ex instanceof HttpMessageNotReadableException theEx) {
      return handleHttpMessageNotReadable(theEx, headers, HttpStatus.BAD_REQUEST, request);
    }
    ...
  }

  // 处理某些异常时把异常信息转换为 ProblemDetail
  protected ResponseEntity<Object> handleHttpMessageNotReadable(HttpMessageNotReadableException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request) {
    ProblemDetail body = createProblemDetail(ex, status, "Failed to read request", null, null, request);
    return handleExceptionInternal(ex, body, headers, status, request);
  }

  // 处理部分异常时，如果没有 body 且异常类型为 ErrorResponse 时，根据该异常生成 ProblemDetail
  protected ResponseEntity<Object> handleExceptionInternal(Exception ex, @Nullable Object body, HttpHeaders headers, HttpStatusCode statusCode, WebRequest request) {
    ...
    if (body == null && ex instanceof ErrorResponse errorResponse) {
      body = errorResponse.updateAndGetBody(this.messageSource, LocaleContextHolder.getLocale());
    }

    return createResponseEntity(body, headers, statusCode, request);
  }
}
```

我们自己的应用可以自定义异常实现 `org.springframework.web.ErrorResponse` 接口，即可返回符合规范的异常描述信息。

-----

还有一个特殊点是，`ProblemDetail` 在处理自定义属性时，为把自定义属性放到最外层，没有直接使用 Jackson 的注解，而是通过 `ProblemDetailJacksonMixin` 实现，把未知的属性放到 `properties` 中，这样可以把类与 Jackson 解耦。

“MixIn” 是 Jackson 中较少使用的一个功能点，可以通过 `ObjectMapper#addMixIn`注入自定义的“MixIn”，允许修改指定类型对象的序列化/反序列化方式。

```java
@JsonInclude(NON_EMPTY)
public interface ProblemDetailJacksonMixin {

  @JsonAnySetter
  void setProperty(String name, @Nullable Object value);

  @JsonAnyGetter
  Map<String, Object> getProperties();

}
```

小补充： `@ExceptionHandler` 注解由 `ExceptionHandlerExceptionResolver` 通过 `ControllerAdviceBean`（封装 `@ControllerAdvice` 定义的信息）搜索 `@ExceptionHandler` 标注的方法并缓存，等发生异常时调用自定义的异常拦截方法。
