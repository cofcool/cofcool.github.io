---
layout: post
category : Tech
title : Spring CORS 源码解析
tags : [spring, java, sourcecode]
excerpt: 现代 Web 开发一般都会采用前后端分离的方式，在这种情况下跨域成为了开发中需要解决的一个重要问题，我们来看看 Spring web MVC 是如何处理该问题
---
{% include JB/setup %}

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. CORS 是什么](#1-cors-是什么)
- [2. Spring web MVC 如何处理跨域](#2-spring-web-mvc-如何处理跨域)
- [3. Spring Security 如何处理跨域](#3-spring-security-如何处理跨域)

<!-- /code_chunk_output -->


## 1. CORS 是什么

`CORS` 全称为 [Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)，为了安全浏览器会限制非同源的请求除非该请求得到允许。对于简单请求只要服务器返回正确的响应头即可，但是有些请求需要先进行预检请求，要求必须首先使用 `OPTIONS` 方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求。

> 跨域资源共享(CORS) 是一种机制，它使用额外的 HTTP 头来告诉浏览器  让运行在一个 origin (domain) 上的Web应用被准许访问来自不同源服务器上的指定的资源。当一个资源从与该资源本身所在的服务器不同的域、协议或端口请求一个资源时，资源会发起一个跨域 HTTP 请求。
> 跨域资源共享（ CORS ）机制允许 Web 应用服务器进行跨域访问控制，从而使跨域数据传输得以安全进行。现代浏览器支持在 API 容器中（例如 XMLHttpRequest 或 Fetch ）使用 CORS，以降低跨域 HTTP 请求所带来的风险。

## 2. Spring web MVC 如何处理跨域

web 请求处理流程简图:

<!--
```plantuml
FrameworkServlet -> FrameworkServlet: doOptions
FrameworkServlet -> DispatcherServlet: doService
DispatcherServlet -> DispatcherServlet: doDispatch
DispatcherServlet -> HandlerMapping: getHandler
HandlerMapping -> HandlerMapping: getHandler
HandlerMapping -> DispatcherServlet: HandlerExecutionChain
DispatcherServlet -> HandlerAdapter: getHandlerAdapter
HandlerAdapter -> HandlerAdapter: handle
HandlerAdapter -> DispatcherServlet
DispatcherServlet -> FrameworkServlet
```
-->

<svg xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="413px" preserveAspectRatio="none" style="width:608px;height:413px;" version="1.1" viewBox="0 0 608 413" width="608px" zoomAndPan="magnify"><defs><filter height="300%" id="f1b6mi2bdx7h6g" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="78" x2="78" y1="38.4883" y2="372.9727"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="227" x2="227" y1="38.4883" y2="372.9727"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="398.5" x2="398.5" y1="38.4883" y2="372.9727"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="537.5" x2="537.5" y1="38.4883" y2="372.9727"></line><rect fill="#FEFECE" filter="url(#f1b6mi2bdx7h6g)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="137" x="8" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="123" x="15" y="23.5352">FrameworkServlet</text><rect fill="#FEFECE" filter="url(#f1b6mi2bdx7h6g)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="137" x="8" y="371.9727"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="123" x="15" y="392.5078">FrameworkServlet</text><rect fill="#FEFECE" filter="url(#f1b6mi2bdx7h6g)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="133" x="159" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="119" x="166" y="23.5352">DispatcherServlet</text><rect fill="#FEFECE" filter="url(#f1b6mi2bdx7h6g)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="133" x="159" y="371.9727"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="119" x="166" y="392.5078">DispatcherServlet</text><rect fill="#FEFECE" filter="url(#f1b6mi2bdx7h6g)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="128" x="332.5" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="114" x="339.5" y="23.5352">HandlerMapping</text><rect fill="#FEFECE" filter="url(#f1b6mi2bdx7h6g)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="128" x="332.5" y="371.9727"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="114" x="339.5" y="392.5078">HandlerMapping</text><rect fill="#FEFECE" filter="url(#f1b6mi2bdx7h6g)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="123" x="474.5" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="109" x="481.5" y="23.5352">HandlerAdapter</text><rect fill="#FEFECE" filter="url(#f1b6mi2bdx7h6g)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="123" x="474.5" y="371.9727"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="109" x="481.5" y="392.5078">HandlerAdapter</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="78.5" x2="120.5" y1="69.7988" y2="69.7988"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="120.5" x2="120.5" y1="69.7988" y2="82.7988"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="79.5" x2="120.5" y1="82.7988" y2="82.7988"></line><polygon fill="#A80036" points="89.5,78.7988,79.5,82.7988,89.5,86.7988,85.5,82.7988" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="66" x="85.5" y="65.0566">doOptions</text><polygon fill="#A80036" points="215.5,108.1094,225.5,112.1094,215.5,116.1094,219.5,112.1094" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="78.5" x2="221.5" y1="112.1094" y2="112.1094"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="60" x="85.5" y="107.3672">doService</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="227.5" x2="269.5" y1="141.4199" y2="141.4199"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="269.5" x2="269.5" y1="141.4199" y2="154.4199"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="228.5" x2="269.5" y1="154.4199" y2="154.4199"></line><polygon fill="#A80036" points="238.5,150.4199,228.5,154.4199,238.5,158.4199,234.5,154.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="72" x="234.5" y="136.6777">doDispatch</text><polygon fill="#A80036" points="386.5,179.7305,396.5,183.7305,386.5,187.7305,390.5,183.7305" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="227.5" x2="392.5" y1="183.7305" y2="183.7305"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="69" x="234.5" y="178.9883">getHandler</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="398.5" x2="440.5" y1="213.041" y2="213.041"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="440.5" x2="440.5" y1="213.041" y2="226.041"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="399.5" x2="440.5" y1="226.041" y2="226.041"></line><polygon fill="#A80036" points="409.5,222.041,399.5,226.041,409.5,230.041,405.5,226.041" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="69" x="405.5" y="208.2988">getHandler</text><polygon fill="#A80036" points="238.5,251.3516,228.5,255.3516,238.5,259.3516,234.5,255.3516" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="232.5" x2="397.5" y1="255.3516" y2="255.3516"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="147" x="244.5" y="250.6094">HandlerExecutionChain</text><polygon fill="#A80036" points="526,280.6621,536,284.6621,526,288.6621,530,284.6621" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="227.5" x2="532" y1="284.6621" y2="284.6621"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="118" x="234.5" y="279.9199">getHandlerAdapter</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="538" x2="580" y1="313.9727" y2="313.9727"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="580" x2="580" y1="313.9727" y2="326.9727"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="539" x2="580" y1="326.9727" y2="326.9727"></line><polygon fill="#A80036" points="549,322.9727,539,326.9727,549,330.9727,545,326.9727" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="42" x="545" y="309.2305">handle</text><polygon fill="#A80036" points="238.5,336.9727,228.5,340.9727,238.5,344.9727,234.5,340.9727" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="232.5" x2="537" y1="340.9727" y2="340.9727"></line><polygon fill="#A80036" points="89.5,350.9727,79.5,354.9727,89.5,358.9727,85.5,354.9727" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="83.5" x2="226.5" y1="354.9727" y2="354.9727"></line></g></svg>

当跨域时的预检请求(OPTIONS)发生时，`HandlerMapping.getHandler` 便会返回 `PreFlightHandler` 作为本次请求的"handler"，它实现了 `HttpRequestHandler` 接口，因此由 `HttpRequestHandlerAdapter`处理。

调用流程简图：

<!--
```plantuml
PreFlightHandler -> PreFlightHandler: handleRequest
PreFlightHandler -> CorsProcessor: processRequest
CorsProcessor -> CorsConfiguration: checkOrigin,checkHeaders...
CorsConfiguration -> CorsProcessor
CorsProcessor -> PreFlightHandler
```
-->

<svg xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="227px" preserveAspectRatio="none" style="width:493px;height:227px;" version="1.1" viewBox="0 0 493 227" width="493px" zoomAndPan="magnify"><defs><filter height="300%" id="f1uj7exaznlybl" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="74" x2="74" y1="38.4883" y2="187.4199"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="209" x2="209" y1="38.4883" y2="187.4199"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="413.5" x2="413.5" y1="38.4883" y2="187.4199"></line><rect fill="#FEFECE" filter="url(#f1uj7exaznlybl)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="129" x="8" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="115" x="15" y="23.5352">PreFlightHandler</text><rect fill="#FEFECE" filter="url(#f1uj7exaznlybl)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="129" x="8" y="186.4199"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="115" x="15" y="206.9551">PreFlightHandler</text><rect fill="#FEFECE" filter="url(#f1uj7exaznlybl)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="113" x="151" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="99" x="158" y="23.5352">CorsProcessor</text><rect fill="#FEFECE" filter="url(#f1uj7exaznlybl)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="113" x="151" y="186.4199"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="99" x="158" y="206.9551">CorsProcessor</text><rect fill="#FEFECE" filter="url(#f1uj7exaznlybl)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="142" x="340.5" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="128" x="347.5" y="23.5352">CorsConfiguration</text><rect fill="#FEFECE" filter="url(#f1uj7exaznlybl)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="142" x="340.5" y="186.4199"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="128" x="347.5" y="206.9551">CorsConfiguration</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="74.5" x2="116.5" y1="69.7988" y2="69.7988"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="116.5" x2="116.5" y1="69.7988" y2="82.7988"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="75.5" x2="116.5" y1="82.7988" y2="82.7988"></line><polygon fill="#A80036" points="85.5,78.7988,75.5,82.7988,85.5,86.7988,81.5,82.7988" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="92" x="81.5" y="65.0566">handleRequest</text><polygon fill="#A80036" points="197.5,108.1094,207.5,112.1094,197.5,116.1094,201.5,112.1094" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="74.5" x2="203.5" y1="112.1094" y2="112.1094"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="99" x="81.5" y="107.3672">processRequest</text><polygon fill="#A80036" points="401.5,137.4199,411.5,141.4199,401.5,145.4199,405.5,141.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="209.5" x2="407.5" y1="141.4199" y2="141.4199"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="180" x="216.5" y="136.6777">checkOrigin,checkHeaders...</text><polygon fill="#A80036" points="220.5,151.4199,210.5,155.4199,220.5,159.4199,216.5,155.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="214.5" x2="412.5" y1="155.4199" y2="155.4199"></line><polygon fill="#A80036" points="85.5,165.4199,75.5,169.4199,85.5,173.4199,81.5,169.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="79.5" x2="208.5" y1="169.4199" y2="169.4199"></line></g></svg>

从上已经了解到了基本调用流程，那具体代码是如何执行的呢?

`AbstractHandlerMapping`，`HandlerMapping` 的抽象类实现，“Spring web MVC” 中的大部分 `HandlerMapping` 实现类都是继承本类，因此跨域处理相关代码从本类开始。

```java
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    Object handler = getHandlerInternal(request);
    
    ...

    // 如果是跨域预检请求则创建对应的 HandlerExecutionChain
    if (hasCorsConfigurationSource(handler) || CorsUtils.isPreFlightRequest(request)) {
        // 如果配置了 corsConfigurationSource 则根据它获取 CorsConfiguration
        CorsConfiguration config = (this.corsConfigurationSource != null ? this.corsConfigurationSource.getCorsConfiguration(request) : null);
        // 获取 handler 携带的 CorsConfiguration
        //
        // 注意:
        // AbstractHandlerMethodMapping 定义 initCorsConfiguration 方法, 以及由内部类 MappingRegistry
        // 负责处理和缓存 initCorsConfiguration 方法返回的配置,
        // 然后在 getCorsConfiguration 方法中把该配置和父类该方法返回的配置进行合并,
        //  可通过 WebMvcConfigurer.addCorsMappings 方法进行配置 MappingRegistry
        // 子类 RequestMappingHandlerMapping 在 initCorsConfiguration 方法中处理 CrossOrigin 注解并创建对应的 CorsConfiguration
        CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
        // 合并跨域配置
        config = (config != null ? config.combine(handlerConfig) : handlerConfig);
        // 配置 HandlerExecutionChain
        executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
    }

    return executionChain;
}

// 配置 HandlerExecutionChain
protected HandlerExecutionChain getCorsHandlerExecutionChain(HttpServletRequest request,
        HandlerExecutionChain chain, @Nullable CorsConfiguration config) {

    if (CorsUtils.isPreFlightRequest(request)) {
        HandlerInterceptor[] interceptors = chain.getInterceptors();
        // 如果是预检请求的话创建 HandlerExecutionChain
        chain = new HandlerExecutionChain(new PreFlightHandler(config), interceptors);
    }
    else {
        // 如果不是预检请求，则添加 CorsInterceptor
        // CorsInterceptor 同样也通过 CorsProcessor 处理跨域请求，因此本文不做过多说明
        chain.addInterceptor(0, new CorsInterceptor(config));
    }
    return chain;
}
```

`CorsProcessor` 默认实现类为 `DefaultCorsProcessor`，根据 `CorsConfiguration` 进行请求处理，基本上对 `CorsConfiguration` 进行封装以及添加跨域相关响应头等方面的处理，具体的跨域相关检查由 `CorsConfiguration` 完成，下面简单分析 `CorsConfiguration` 对“origin”的检查，其它同理就不做过多介绍，感兴趣的朋友可直接查看源码。

```java
// 检查 origin, 返回 null 表示检查不通过
public String checkOrigin(@Nullable String requestOrigin) {
    if (!StringUtils.hasText(requestOrigin)) {
        return null;
    }
    // 配置的 allowedOrigins 是否为 null
    if (ObjectUtils.isEmpty(this.allowedOrigins)) {
        return null;
    }

    // allowedOrigins 为 * 时, 如果允许请求携带 cookie, 则返回请求来源地址, 否则返回 *
    // 发出请求时，如果前端携带了 cookie, 而服务器配置为 *, 浏览器则会拒绝请求
    if (this.allowedOrigins.contains(ALL)) {
        if (this.allowCredentials != Boolean.TRUE) {
            return ALL;
        }
        else {
            return requestOrigin;
        }
    }
    // 遍历配置的 allowedOrigins, 如果包含请求来源地址则返回该地址
    for (String allowedOrigin : this.allowedOrigins) {
        if (requestOrigin.equalsIgnoreCase(allowedOrigin)) {
            return requestOrigin;
        }
    }

    return null;
}
```

除了可以通过 `PreFlightHandler`，`CorsFilter` 和 `CrossOrigin` 处理跨域外，框架还提供了 `CorsFilter` 来处理跨域请求，该类在默认情况下并未启用，不过 `Spring Security` 启用了该类，详情参考下文所述。

## 3. Spring Security 如何处理跨域

使用 `Spring Security` 时一般会对 `HttpSecurity` 进行配置，如下所示：

```java
@Configuration
@EnableWebSecurity
public class MyWebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            .antMatchers("/public/**")
            .permitAll()
            .and()
            // 允许跨域
            .cors();
    }
}
```

如此简单方法就配置好允许跨域了，其中到底发生了什么，让我们一探究竟。

`cors` 方法代码如下：

```java
public CorsConfigurer<HttpSecurity> cors() throws Exception {
    return getOrApply(new CorsConfigurer<>());
}
```

调用`cors`方法时会获取到`CorsConfigurer`，该类负责创建和配置 `CorsFilter`。

```java
// 它实现 SecurityConfigurer 接口定义的方法
// SecurityBuilder 在 build 时会调用，参考 AbstractConfiguredSecurityBuilder
public void configure(H http) {
    ApplicationContext context = http.getSharedObject(ApplicationContext.class);

    // 获取 CorsFilter
    CorsFilter corsFilter = getCorsFilter(context);
    if (corsFilter == null) {
        throw new IllegalStateException(
                "Please configure either a " + CORS_FILTER_BEAN_NAME + " bean or a "
                        + CORS_CONFIGURATION_SOURCE_BEAN_NAME + "bean.");
    }
    // 添加 CorsFilter 到 filter 调用链
    http.addFilter(corsFilter);
}

private CorsFilter getCorsFilter(ApplicationContext context) {
    if (this.configurationSource != null) {
        // 根据 corsConfigurationSource 创建
        return new CorsFilter(this.configurationSource);
    }

    // 判断容器中是否包含名字为 corsFilter 的实例
    // 如果包含则直接返回
    boolean containsCorsFilter = context
            .containsBeanDefinition(CORS_FILTER_BEAN_NAME);
    if (containsCorsFilter) {
        return context.getBean(CORS_FILTER_BEAN_NAME, CorsFilter.class);
    }

    // 判断容器中是否包含名字为 corsConfigurationSource 的实例
    // 如果包含则根据该实例创建 CorsFilter
    boolean containsCorsSource = context
            .containsBean(CORS_CONFIGURATION_SOURCE_BEAN_NAME);
    if (containsCorsSource) {
        CorsConfigurationSource configurationSource = context.getBean(
                CORS_CONFIGURATION_SOURCE_BEAN_NAME, CorsConfigurationSource.class);
        return new CorsFilter(configurationSource);
    }

    // 判断当前类加载器是否加载 org.springframework.web.servlet.handler.HandlerMappingIntrospector 类
    // 如果是则获取 HandlerMappingIntrospector 实例，并根据它创建 CorsFilter
    boolean mvcPresent = ClassUtils.isPresent(HANDLER_MAPPING_INTROSPECTOR,
            context.getClassLoader());
    if (mvcPresent) {
        return MvcCorsFilter.getMvcCorsFilter(context);
    }
    return null;
}
```

从以上代码可以了解到四种获取 `CorsFilter` 的方式：

* 根据配置的 `CorsConfigurationSource` 创建
* 从容器中读取 `CorsFilter`
* 从容器中读取  `CorsConfigurationSource` 并以此创建
* 从容器中读取  `HandlerMappingIntrospector` 并以此创建

默认使用第四种方式，该类可调用"MVC"中对跨域的配置，如通过 `WebMvcConfigurer.addCorsMappings` 所做的配置。如果想要自定义配置的话使用第三种方式即创建 `CorsConfigurationSource` 最为简单。

以上四种处理方式都是通过 `CorsFilter` 进行跨域处理，它也是调用 `CorsProcessor.processRequest` 来处理的，和前文所述相同就不详细说明来。它的构造方法只有一个参数，为 `CorsConfigurationSource` 类型。`CorsConfigurationSource` 负责从请求中读取对应的“CorsConfiguration”，应用中常用的实现类为`UrlBasedCorsConfigurationSource`，该类可根据路径来配置，如 `source.registerCorsConfiguration("/**", configuration)`，把自定义的"configuration"注册到全局路径。

```java
private final CorsConfigurationSource configSource;

private CorsProcessor processor = new DefaultCorsProcessor();

@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
        FilterChain filterChain) throws ServletException, IOException {

    // 获取 CorsConfiguration
    CorsConfiguration corsConfiguration = this.configSource.getCorsConfiguration(request);
    // 检查请求
    boolean isValid = this.processor.processRequest(corsConfiguration, request, response);
    // 如果检查不符合跨域规范则直接返回中断本次请求
    if (!isValid || CorsUtils.isPreFlightRequest(request)) {
        return;
    }
    filterChain.doFilter(request, response);
}
```
