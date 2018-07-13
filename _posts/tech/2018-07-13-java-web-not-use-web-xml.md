---
layout: post
category: Tech
title: 是时候抛弃web.xml了?
tags: [java]
excerpt: Servlet 3.0 规定只要Web项目中实现了ServletContainerInitializer接口，即可不通过web.xml注册Servlet和Filter。
---

{% include JB/setup %}

Servlet 3 规范规定了只要Web项目中实现了`ServletContainerInitializer`接口，即可不通过web.xml注册Servlet和Filter。当使用代码的方式配置时，Web容器会寻找应用中实现了ServletContainerInitializer接口的类，并调用`onStartup`方法初始化应用。

如下所述:

> The following methods are added to ServletContext since Servlet 3.0 to enable programmatic definition of servlets, filters and the url pattern that they map to. These methods can only be called during the initialization of the application either from the contexInitialized method of a ServletContextListenerimplementation or from the onStartup method of a ServletContainerInitializer implementation. In addition to adding Servlets and Filters, one can also look up an instance of a Registration object corresponding to a Servlet or Filter or a map of all the Registration objects for the Servlets or Filters. If the ServletContext passed to the ServletContextListener ’s contextInitialized method was neither declared in web.xml or web-fragment.xml nor annotated with @WebListener then an UnsupportedOperationException MUST be thrown for all the methods defined for programmatic configuration of servlets, filters and listeners.
>
> --- Java ™ Servlet Specification, Servlet 3.0

目录:


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [1. 代码编写](#1-代码编写)
* [2. 实现分析](#2-实现分析)
	* [2.1 Tomcat](#21-tomcat)
	* [2.2 Spring MVC](#22-spring-mvc)

<!-- /code_chunk_output -->


## 1. 代码编写

下面我们来看看如何在项目中使用这一新特性。

Spring MVC的`SpringServletContainerInitializer`实现了`ServletContainerInitializer`接口，因此可通过Java代码来配置Servlet和Filter等类。


在Spring MVC项目中，继承`AbstractAnnotationConfigDispatcherServletInitializer`即可。

```java
/**
 * @author CofCool
 * @date 2018/7/13
 */
public class SpringWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    /**
     * 上下文配置，
     * 也就是传web.xml中的ContextLoaderListener配置
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[] { RootConfigure.class };
    }

    /**
     * MVC配置
     * 也就是web.xml中的的DispatcherServlet配置
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[] { WebConfigure.class };
    }

    /**
     * DispatcherServlet映射路径配置
     */
    @Override
    protected String[] getServletMappings() {
        return new String[] { "/*" };
    }

    /**
     * 注册Filter
     */
    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] { new TestFilter() };
    }
}
```

**RootConfigure**: 配置包扫描路径等。

```java
/**
 * @author CofCool
 * @date 2018/7/13
 */
@Configuration
@ComponentScan(basePackages = {"net.cofcool.test.spring"})
public class RootConfigure {

}
```

**WebConfigure**: MVC相关配置，Controller扫描，视图解析等。

```java
/**
 * @author CofCool
 * @date 2018/7/13
 */
@Configuration
@EnableWebMvc
@ComponentScan("net.cofcool.test.spring.app")
public class WebConfigure extends WebMvcConfigurerAdapter{


    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/view/");
        resolver.setSuffix(".jsp");

        return resolver;
    }

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

通过以上简单代码即可实现Spring MVC应用的基础配置，是不是比`web.xml`的方式更为简单容易！

完整示例可查看[SpringTest](https://github.com/cofcool/SpringTest/tree/master/SpringTest-servlet3)。

## 2. 实现分析

### 2.1 Tomcat

以下代码以Tomcat9源码为例。

ContextConfig类负责扫描和添加ServletContainerInitializer。

```java
/**
 * 扫描应用中ServletContainerInitializer的实现类
 */
protected void processServletContainerInitializers() {

    List<ServletContainerInitializer> detectedScis;
    try {
        WebappServiceLoader<ServletContainerInitializer> loader = new WebappServiceLoader<>(context);
        detectedScis = loader.load(ServletContainerInitializer.class);
    } catch (IOException e) {
        log.error(sm.getString(
                "contextConfig.servletContainerInitializerFail",
                context.getName()),
            e);
        ok = false;
        return;
    }

    // 缓存扫描到的实现类
    for (ServletContainerInitializer sci : detectedScis) {
        initializerClassMap.put(sci, new HashSet<Class<?>>());

        HandlesTypes ht;
        try {
            ht = sci.getClass().getAnnotation(HandlesTypes.class);
        } catch (Exception e) {
            if (log.isDebugEnabled()) {
                log.info(sm.getString("contextConfig.sci.debug",
                        sci.getClass().getName()),
                        e);
            } else {
                log.info(sm.getString("contextConfig.sci.info",
                        sci.getClass().getName()));
            }
            continue;
        }
        if (ht == null) {
            continue;
        }
        Class<?>[] types = ht.value();
        if (types == null) {
            continue;
        }

        for (Class<?> type : types) {
            if (type.isAnnotation()) {
                handlesTypesAnnotations = true;
            } else {
                handlesTypesNonAnnotations = true;
            }
            Set<ServletContainerInitializer> scis =
                    typeInitializerMap.get(type);
            if (scis == null) {
                scis = new HashSet<>();
                typeInitializerMap.put(type, scis);
            }
            scis.add(sci);
        }
    }
}

// 项目启动配置时调用，一般在START_EVENT之前，BEFORE_START_EVENT之后
// Lifecycle.CONFIGURE_START_EVENT
protected void webConfig() {
    ...
    // Step 11. 添加ServletContainerInitializer到上下文中
        if (ok) {
            for (Map.Entry<ServletContainerInitializer,
                    Set<Class<?>>> entry :
                        initializerClassMap.entrySet()) {
                if (entry.getValue().isEmpty()) {
                    context.addServletContainerInitializer(
                            entry.getKey(), null);
                } else {
                    context.addServletContainerInitializer(
                            entry.getKey(), entry.getValue());
                }
            }
        }
}
```

Context的实现为`StandardContext`。

```java
// 调用onStartup方法
protected synchronized void startInternal() throws LifecycleException {
    ...
    for (Map.Entry<ServletContainerInitializer, Set<Class<?>>> entry :
        initializers.entrySet()) {
        try {
            entry.getKey().onStartup(entry.getValue(),
                    getServletContext());
        } catch (ServletException e) {
            log.error(sm.getString("standardContext.sciFail"), e);
            ok = false;
            break;
        }
    }
    ...
}
```

### 2.2 Spring MVC

SpringServletContainerInitializer实现了ServletContainerInitializer接口，并通过`@HandlesTypes`声明可以处理的类为`WebApplicationInitializer`。

AbstractContextLoaderInitializer实现了WebApplicationInitializer，并定义了`createRootApplicationContext`，负责创建ContextLoaderListener。它的子类AbstractDispatcherServletInitializer定义了`createServletApplicationContext`，负责创建DispatcherServlet。

AbstractAnnotationConfigDispatcherServletInitializer继承自AbstractDispatcherServletInitializer，实现了`createRootApplicationContext`和`createServletApplicationContext`，并定义了getRootConfigClasses和getServletConfigClasses，也即是我们上文中项目配置类中重写的两个方法。

<!--?xml version="1.0" encoding="UTF-8" standalone="no"?--><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="351px" preserveAspectRatio="none" style="width:346px;height:351px;" version="1.1" viewBox="0 0 346 351" width="346px" zoomAndPan="magnify"><defs><filter height="300%" id="f1309qykcsr1zz" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><text fill="#000000" font-family="sans-serif" font-size="18" lengthAdjust="spacingAndGlyphs" textLength="36" x="158.5" y="17.4023">类图</text><!--class WebApplicationInitializer--><rect fill="#FEFECE" filter="url(#f1309qykcsr1zz)" height="60.9551" id="WebApplicationInitializer" style="stroke: #A80036; stroke-width: 1.5;" width="248" x="46.5" y="29.6992"></rect><ellipse cx="95.75" cy="45.6992" fill="#B4A7E5" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M91.6777,41.4644 L91.6777,39.3062 L99.0571,39.3062 L99.0571,41.4644 L96.5918,41.4644 L96.5918,49.541 L99.0571,49.541 L99.0571,51.6992 L91.6777,51.6992 L91.6777,49.541 L94.1431,49.541 L94.1431,41.4644 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" font-style="italic" lengthAdjust="spacingAndGlyphs" textLength="141" x="116.25" y="50.2344">WebApplicationInitializer</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="47.5" x2="293.5" y1="61.6992" y2="61.6992"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="47.5" x2="293.5" y1="69.6992" y2="69.6992"></line><ellipse cx="57.5" cy="81.6768" fill="#84BE84" rx="3" ry="3" style="stroke: #038048; stroke-width: 1.0;"></ellipse><text fill="#000000" font-family="sans-serif" font-size="11" lengthAdjust="spacingAndGlyphs" textLength="222" x="66.5" y="84.334">onStartup(ServletContext servletContext)</text><!--class AbstractContextLoaderInitializer--><rect fill="#FEFECE" filter="url(#f1309qykcsr1zz)" height="48" id="AbstractContextLoaderInitializer" style="stroke: #A80036; stroke-width: 1.5;" width="216" x="62.5" y="126.1992"></rect><ellipse cx="77.5" cy="142.1992" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M80.4731,147.8423 Q79.8921,148.1411 79.2529,148.2905 Q78.6138,148.4399 77.9082,148.4399 Q75.4014,148.4399 74.0815,146.7881 Q72.7617,145.1362 72.7617,142.0151 Q72.7617,138.8857 74.0815,137.2339 Q75.4014,135.582 77.9082,135.582 Q78.6138,135.582 79.2612,135.7314 Q79.9087,135.8809 80.4731,136.1797 L80.4731,138.9023 Q79.8423,138.3213 79.2488,138.0515 Q78.6553,137.7817 78.0244,137.7817 Q76.6797,137.7817 75.9949,138.8484 Q75.3101,139.915 75.3101,142.0151 Q75.3101,144.1069 75.9949,145.1736 Q76.6797,146.2402 78.0244,146.2402 Q78.6553,146.2402 79.2488,145.9705 Q79.8423,145.7007 80.4731,145.1196 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="184" x="91.5" y="146.7344">AbstractContextLoaderInitializer</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="63.5" x2="277.5" y1="158.1992" y2="158.1992"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="63.5" x2="277.5" y1="166.1992" y2="166.1992"></line><!--class AbstractDispatcherServletInitializer--><rect fill="#FEFECE" filter="url(#f1309qykcsr1zz)" height="48" id="AbstractDispatcherServletInitializer" style="stroke: #A80036; stroke-width: 1.5;" width="232" x="54.5" y="209.1992"></rect><ellipse cx="69.5" cy="225.1992" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M72.4731,230.8423 Q71.8921,231.1411 71.2529,231.2905 Q70.6138,231.4399 69.9082,231.4399 Q67.4014,231.4399 66.0815,229.7881 Q64.7617,228.1362 64.7617,225.0151 Q64.7617,221.8857 66.0815,220.2339 Q67.4014,218.582 69.9082,218.582 Q70.6138,218.582 71.2612,218.7314 Q71.9087,218.8809 72.4731,219.1797 L72.4731,221.9023 Q71.8423,221.3213 71.2488,221.0515 Q70.6553,220.7817 70.0244,220.7817 Q68.6797,220.7817 67.9949,221.8484 Q67.3101,222.915 67.3101,225.0151 Q67.3101,227.1069 67.9949,228.1736 Q68.6797,229.2402 70.0244,229.2402 Q70.6553,229.2402 71.2488,228.9705 Q71.8423,228.7007 72.4731,228.1196 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="200" x="83.5" y="229.7344">AbstractDispatcherServletInitializer</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="55.5" x2="285.5" y1="241.1992" y2="241.1992"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="55.5" x2="285.5" y1="249.1992" y2="249.1992"></line><!--class AbstractAnnotationConfigDispatcherServletInitializer--><rect fill="#FEFECE" filter="url(#f1309qykcsr1zz)" height="48" id="AbstractAnnotationConfigDispatcherServletInitializer" style="stroke: #A80036; stroke-width: 1.5;" width="329" x="6" y="292.1992"></rect><ellipse cx="21" cy="308.1992" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M23.9731,313.8423 Q23.3921,314.1411 22.7529,314.2905 Q22.1138,314.4399 21.4082,314.4399 Q18.9014,314.4399 17.5815,312.7881 Q16.2617,311.1362 16.2617,308.0151 Q16.2617,304.8857 17.5815,303.2339 Q18.9014,301.582 21.4082,301.582 Q22.1138,301.582 22.7612,301.7314 Q23.4087,301.8809 23.9731,302.1797 L23.9731,304.9023 Q23.3423,304.3213 22.7488,304.0515 Q22.1553,303.7817 21.5244,303.7817 Q20.1797,303.7817 19.4949,304.8484 Q18.8101,305.915 18.8101,308.0151 Q18.8101,310.1069 19.4949,311.1736 Q20.1797,312.2402 21.5244,312.2402 Q22.1553,312.2402 22.7488,311.9705 Q23.3423,311.7007 23.9731,311.1196 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="297" x="35" y="312.7344">AbstractAnnotationConfigDispatcherServletInitializer</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="7" x2="334" y1="324.1992" y2="324.1992"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="7" x2="334" y1="332.1992" y2="332.1992"></line><!--link WebApplicationInitializer to AbstractContextLoaderInitializer--><path d="M170.5,110.824 C170.5,115.8631 170.5,120.9022 170.5,125.9414 " fill="none" id="WebApplicationInitializer-AbstractContextLoaderInitializer" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="163.5001,110.7591,170.5,90.759,177.5001,110.759,163.5001,110.7591" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link AbstractContextLoaderInitializer to AbstractDispatcherServletInitializer--><path d="M170.5,194.6453 C170.5,199.3911 170.5,204.137 170.5,208.8828 " fill="none" id="AbstractContextLoaderInitializer-AbstractDispatcherServletInitializer" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="163.5001,194.4979,170.5,174.4979,177.5001,194.4978,163.5001,194.4979" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link AbstractDispatcherServletInitializer to AbstractAnnotationConfigDispatcherServletInitializer--><path d="M170.5,277.6453 C170.5,282.3911 170.5,287.137 170.5,291.8828 " fill="none" id="AbstractDispatcherServletInitializer-AbstractAnnotationConfigDispatcherServletInitializer" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="163.5001,277.4979,170.5,257.4979,177.5001,277.4978,163.5001,277.4979" style="stroke: #A80036; stroke-width: 1.0;"></polygon></g></svg>

<!--
```plantuml
title: 类图
left to right direction

interface WebApplicationInitializer {
    + onStartup(ServletContext servletContext)
}

WebApplicationInitializer <|- AbstractContextLoaderInitializer
AbstractContextLoaderInitializer <|-  AbstractDispatcherServletInitializer
AbstractDispatcherServletInitializer <|-  AbstractAnnotationConfigDispatcherServletInitializer
```
-->