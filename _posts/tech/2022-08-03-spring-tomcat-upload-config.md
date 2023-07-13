---
layout: post
category : Tech
title : Spring Boot 如何限制上传文件大小
tags : [java, spring, tomcat]
excerpt: spring.servlet.multipart.max-file-size 是如何转变为 Tomcat 配置的？
---
{% include JB/setup %}

Spring Boot 项目中会使用 `spring.servlet.multipart.max-file-size` 来修改上传文件大小，默认为 1MB。那它是如何影响 Tomcat 的？

Servlet 规范中上传文件相关定义：

* `javax.servlet.ServletRegistration.Dynamic#setMultipartConfig`
* `javax.servlet.MultipartConfigElement`
* `javax.servlet.http.HttpServletRequest#getParts`

Tomcat 当然也实现了该规范，流程看下图

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentStyleType="text/css" height="221px" preserveAspectRatio="none" style="width:606px;height:221px;background:#FFFFFF;" version="1.1" viewBox="0 0 606 221" width="606px" zoomAndPan="magnify"><defs/><g><line style="stroke:#181818;stroke-width:0.5;stroke-dasharray:5.0,5.0;" x1="124" x2="124" y1="36.2969" y2="185.8281"/><line style="stroke:#181818;stroke-width:0.5;stroke-dasharray:5.0,5.0;" x1="324" x2="324" y1="36.2969" y2="185.8281"/><line style="stroke:#181818;stroke-width:0.5;stroke-dasharray:5.0,5.0;" x1="527.5" x2="527.5" y1="36.2969" y2="185.8281"/><rect fill="#E2E2F0" height="30.2969" rx="2.5" ry="2.5" style="stroke:#181818;stroke-width:0.5;" width="238" x="5" y="5"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="224" x="12" y="24.9951">ApplicationServletRegistration</text><rect fill="#E2E2F0" height="30.2969" rx="2.5" ry="2.5" style="stroke:#181818;stroke-width:0.5;" width="238" x="5" y="184.8281"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="224" x="12" y="204.8232">ApplicationServletRegistration</text><rect fill="#E2E2F0" height="30.2969" rx="2.5" ry="2.5" style="stroke:#181818;stroke-width:0.5;" width="142" x="253" y="5"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="128" x="260" y="24.9951">StandardWrapper</text><rect fill="#E2E2F0" height="30.2969" rx="2.5" ry="2.5" style="stroke:#181818;stroke-width:0.5;" width="142" x="253" y="184.8281"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="128" x="260" y="204.8232">StandardWrapper</text><rect fill="#E2E2F0" height="30.2969" rx="2.5" ry="2.5" style="stroke:#181818;stroke-width:0.5;" width="73" x="491.5" y="5"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="59" x="498.5" y="24.9951">Request</text><rect fill="#E2E2F0" height="30.2969" rx="2.5" ry="2.5" style="stroke:#181818;stroke-width:0.5;" width="73" x="491.5" y="184.8281"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="59" x="498.5" y="204.8232">Request</text><polygon fill="#181818" points="312,63.4297,322,67.4297,312,71.4297,316,67.4297" style="stroke:#181818;stroke-width:1.0;"/><line style="stroke:#181818;stroke-width:1.0;" x1="124" x2="318" y1="67.4297" y2="67.4297"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="116" x="131" y="62.3638">setMultipartConfig</text><line style="stroke:#181818;stroke-width:1.0;" x1="528" x2="570" y1="96.5625" y2="96.5625"/><line style="stroke:#181818;stroke-width:1.0;" x1="570" x2="570" y1="96.5625" y2="109.5625"/><line style="stroke:#181818;stroke-width:1.0;" x1="529" x2="570" y1="109.5625" y2="109.5625"/><polygon fill="#181818" points="539,105.5625,529,109.5625,539,113.5625,535,109.5625" style="stroke:#181818;stroke-width:1.0;"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="64" x="535" y="91.4966">getParts()</text><polygon fill="#181818" points="335,134.6953,325,138.6953,335,142.6953,331,138.6953" style="stroke:#181818;stroke-width:1.0;"/><line style="stroke:#181818;stroke-width:1.0;" x1="329" x2="527" y1="138.6953" y2="138.6953"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="180" x="341" y="133.6294">getMultipartConfigElement()</text><polygon fill="#181818" points="516,163.8281,526,167.8281,516,171.8281,520,167.8281" style="stroke:#181818;stroke-width:1.0;"/><line style="stroke:#181818;stroke-width:1.0;" x1="324" x2="522" y1="167.8281" y2="167.8281"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="149" x="331" y="162.7622">MultipartConfigElement</text><!--SRC=[SomeoCbCJYp9pCyBJYqgoqaj2KfDpomkAG8BAUZQAGIN9EQb91QbX1Sb5XIa5YbO5QUM-9Rcb6GM91QLEEVdfMMcSmMb5fQc5fU0bCEOLkcf9G505SKQciZI6AQbOvZccfEQcvfN0jI7hXZPU0NikW00]--></g></svg>

`Request#getParts` 在解析上传文件时调用 `parseParts`，读取 `MultipartConfigElement` 相关配置，先从 Servlet 容器中获取，如不存在则从 Connector（可以理解为全局配置类） 获取，实现代码如下：

```java
// org.apache.catalina.connector.Request#parseParts
private void parseParts(boolean explicit) {
    ...
    MultipartConfigElement mce = getWrapper().getMultipartConfigElement();

    if (mce == null) {
        if(context.getAllowCasualMultipartParsing()) {
            // 未通过 Dynamic#setMultipartConfig 配置 MultipartConfigElement 时
            // 从 connector 中读取
            mce = new MultipartConfigElement(null, connector.getMaxPostSize(),
                    connector.getMaxPostSize(), connector.getMaxPostSize());
        } else {
            if (explicit) {
                partsParseException = new IllegalStateException(
                        sm.getString("coyoteRequest.noMultipartConfig"));
                return;
            } else {
                parts = Collections.emptyList();
                return;
            }
        }
    }
    ...
}
```

Connector 的配置 `org.apache.catalina.connector.Connector#maxPostSize` 默认为 2MB。


综上，当未配置 MultipartConfigElement 时默认大小为 2MB。

了解到 Tomcat 的配置后再来看 Spring Boot，要修改该配置它只能从两方面入手：

1. 注入 MultipartConfigElement
2. 修改 Connector#maxPostSize

**1. 注入 MultipartConfigElement**

`spring.servlet.multipart.max-file-size` 对应的自动配置类为 `MultipartAutoConfiguration`，它会创建 `javax.servlet.MultipartConfigElement` 实例。接下来 `ServletContextInitializerBeans.ServletRegistrationBeanAdapter#createRegistrationBean` 创建 `RegistrationBean`，它会启动时自动把`MultipartConfigElement`实例注入到 Tomcat 中，具体原理可参考 [是时候抛弃web.xml了? ](https://cofcool.github.io/tech/2018/07/13/java-web-not-use-web-xml)

**2. 修改 Connector#maxPostSize**

配置项为 `server.tomcat.max-http-form-post-size`，由 `TomcatWebServerFactoryCustomizer` 负责把该配置更新到 `Connector#maxPostSize`

---

总之，推荐使用 `spring.servlet.multipart.max-file-size` 配置，它优先级高且普适性更好。