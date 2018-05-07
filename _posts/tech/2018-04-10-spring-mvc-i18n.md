---
layout: post
category : Tech
title : Spring MVC和Hibernate Validator 国际化
tags : [java, shiro]
---
{% include JB/setup %}

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. Spring MVC](#1-spring-mvc)
	* [1.1 配置](#11-配置)
	* [1.2 流程分析](#12-流程分析)
	* [1.3 源码解析](#13-源码解析)
* [2. Hibernate Validator](#2-hibernate-validator)
	* [2.1 配置](#21-配置)
	* [2.2 源码解析](#22-源码解析)

<!-- /code_chunk_output -->


## 1. Spring MVC

### 1.1 配置

**spring.xml**
```xml
<!--
配置语言资源文件
basename： 语言资源文件名
useCodeAsDefaultMessage： 设置为true时，如果没有找到对应的语言字符，则使用传入的code作为返回值
-->
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
    <property name="basename" value="messages"/>
    <property name="useCodeAsDefaultMessage" value="true"/>
    <property name="defaultEncoding" value="UTF-8"/>
</bean>
```

**spring-mvc.xml**
```xml
<!-- 地区解析器，可存储和解析request中的地区信息 -->
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver" />

<!-- 拦截器，可获取request中携带的地区信息，并通过localeResolver来处理 -->
<mvc:interceptors>
	<bean id="localeChangeInterceptor" class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor" />
</mvc:interceptors>
````

### 1.2 流程分析

<!--
```plantuml
调用接口 -> 地区拦截器 : doDispatch
note left: DispatcherServlet
地区拦截器 -> 地区解析器 : postHandle
note left: LocaleChangeInterceptor
地区解析器-> 存储地区: setLocale
note left: LocaleResolver

自定义逻辑处理 -> 地区解析器
地区解析器 -> 根据地区信息获取资源: resolveLocale
根据地区信息获取资源 -> 读取对应区域的资源: getMessage
note left: MessageSource
读取对应区域的资源 -> 自定义逻辑处理
```
-->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="313px" preserveAspectRatio="none" style="width:925px;height:313px;" version="1.1" viewBox="0 0 925 313" width="925px" zoomAndPan="magnify"><defs><filter height="300%" id="f1sgm2io0fnq5z" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="148" x2="148" y1="38.4883" y2="273.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="244" x2="244" y1="38.4883" y2="273.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="342" x2="342" y1="38.4883" y2="273.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="433" x2="433" y1="38.4883" y2="273.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="538" x2="538" y1="38.4883" y2="273.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="685" x2="685" y1="38.4883" y2="273.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="846" x2="846" y1="38.4883" y2="273.041"></line><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="70" x="111" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="56" x="118" y="23.5352">调用接口</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="70" x="111" y="272.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="56" x="118" y="292.5762">调用接口</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="84" x="200" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="70" x="207" y="23.5352">地区拦截器</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="84" x="200" y="272.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="70" x="207" y="292.5762">地区拦截器</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="84" x="298" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="70" x="305" y="23.5352">地区解析器</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="84" x="298" y="272.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="70" x="305" y="292.5762">地区解析器</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="70" x="396" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="56" x="403" y="23.5352">存储地区</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="70" x="396" y="272.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="56" x="403" y="292.5762">存储地区</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="112" x="480" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="98" x="487" y="23.5352">自定义逻辑处理</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="112" x="480" y="272.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="98" x="487" y="292.5762">自定义逻辑处理</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="154" x="606" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="140" x="613" y="23.5352">根据地区信息获取资源</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="154" x="606" y="272.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="140" x="613" y="292.5762">根据地区信息获取资源</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="140" x="774" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="126" x="781" y="23.5352">读取对应区域的资源</text><rect fill="#FEFECE" filter="url(#f1sgm2io0fnq5z)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="140" x="774" y="272.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="126" x="781" y="292.5762">读取对应区域的资源</text><polygon fill="#A80036" points="232,70.4883,242,74.4883,232,78.4883,236,74.4883" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="148" x2="238" y1="74.4883" y2="74.4883"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="72" x="155" y="70.0566">doDispatch</text><path d="M8,53.4883 L8,78.4883 L139,78.4883 L139,63.4883 L129,53.4883 L8,53.4883 " fill="#FBFB77" filter="url(#f1sgm2io0fnq5z)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M129,53.4883 L129,63.4883 L139,63.4883 L129,53.4883 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="110" x="14" y="71.0566">DispatcherServlet</text><polygon fill="#A80036" points="330,109.7988,340,113.7988,330,117.7988,334,113.7988" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="244" x2="336" y1="113.7988" y2="113.7988"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="72" x="251" y="109.3672">postHandle</text><path d="M58,92.7988 L58,117.7988 L235,117.7988 L235,102.7988 L225,92.7988 L58,92.7988 " fill="#FBFB77" filter="url(#f1sgm2io0fnq5z)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M225,92.7988 L225,102.7988 L235,102.7988 L225,92.7988 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="156" x="64" y="110.3672">LocaleChangeInterceptor</text><polygon fill="#A80036" points="421,149.1094,431,153.1094,421,157.1094,425,153.1094" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="342" x2="427" y1="153.1094" y2="153.1094"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="59" x="349" y="148.6777">setLocale</text><path d="M219,132.1094 L219,157.1094 L333,157.1094 L333,142.1094 L323,132.1094 L219,132.1094 " fill="#FBFB77" filter="url(#f1sgm2io0fnq5z)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M323,132.1094 L323,142.1094 L333,142.1094 L323,132.1094 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="93" x="225" y="149.6777">LocaleResolver</text><polygon fill="#A80036" points="353,168.4199,343,172.4199,353,176.4199,349,172.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="347" x2="537" y1="172.4199" y2="172.4199"></line><polygon fill="#A80036" points="673,197.4199,683,201.4199,673,205.4199,677,201.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="342" x2="679" y1="201.4199" y2="201.4199"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="85" x="349" y="196.9883">resolveLocale</text><polygon fill="#A80036" points="834,231.7305,844,235.7305,834,239.7305,838,235.7305" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="685" x2="840" y1="235.7305" y2="235.7305"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="74" x="692" y="231.2988">getMessage</text><path d="M559,214.7305 L559,239.7305 L676,239.7305 L676,224.7305 L666,214.7305 L559,214.7305 " fill="#FBFB77" filter="url(#f1sgm2io0fnq5z)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M666,214.7305 L666,224.7305 L676,224.7305 L666,214.7305 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="96" x="565" y="232.2988">MessageSource</text><polygon fill="#A80036" points="549,251.041,539,255.041,549,259.041,545,255.041" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="543" x2="845" y1="255.041" y2="255.041"></line><!--
@startuml
调用接口 -> 地区拦截器 : doDispatch
note left: DispatcherServlet
地区拦截器 -> 地区解析器 : postHandle
note left: LocaleChangeInterceptor
地区解析器-> 存储地区: setLocale
note left: LocaleResolver

自定义逻辑处理 -> 地区解析器
地区解析器 -> 根据地区信息获取资源: resolveLocale
根据地区信息获取资源 -> 读取对应区域的资源: getMessage
note left: MessageSource
读取对应区域的资源 -> 自定义逻辑处理
@enduml

PlantUML version 1.2018.01(Mon Jan 29 02:08:22 CST 2018)
(GPL source distribution)
Java Runtime: Java(TM) SE Runtime Environment
JVM: Java HotSpot(TM) 64-Bit Server VM
Java Version: 1.8.0_152-b16
Operating System: Mac OS X
OS Version: 10.13.4
Default Encoding: UTF-8
Language: en
Country: CN
--></g></svg>

### 1.3 源码解析

**DispatcherServlet**
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    ...
    mappedHandler = getHandler(processedRequest);
    ...
    // 调用拦截器的预处理方法
    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return;·
    }
    ...
}
```

**LocaleChangeInterceptor**
```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException {
    // 根据paramName属性读取地区信息
    // 注意：改拦截器只会判断请求是否携带地区信息，如果想从请求的HEADER中获取，需重写preHandle方法
    // SessionLocaleResolver的resolveLocale方法已实现从HEADER中获取locale信息
    String newLocale = request.getParameter(getParamName());
    if (newLocale != null) {
        if (checkHttpMethod(request.getMethod())) {
            // 获取 LocaleResolver
            LocaleResolver localeResolver = RequestContextUtils.getLocaleResolver(request);
            if (localeResolver == null) {
                throw new IllegalStateException(
                        "No LocaleResolver found: not in a DispatcherServlet request?");
            }
            try {
                // 调用localeResolver来存储地区信息
                localeResolver.setLocale(request, response, parseLocaleValue(newLocale));
            }
            catch (IllegalArgumentException ex) {
                if (isIgnoreInvalidLocale()) {
                    logger.debug("Ignoring invalid locale value [" + newLocale + "]: " + ex.getMessage());
                }
                else {
                    throw ex;
                }
            }
        }
    }
    // Proceed in any case.
    return true;
}
```

如需从请求头获取地区信息，可参考**MyLocaleChangeInterceptor**的实现。
```java
public class MyLocaleChangeInterceptor extends LocaleChangeInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
        Object handler) throws ServletException {
        String newLocale = request.getParameter(getParamName());

        if (newLocale == null) {
            // SessionLocaleResolver
            // try to get locale message by Accept-Language of request-header or default locale of localeResolver

            LocaleResolver localeResolver = RequestContextUtils.getLocaleResolver(request);

            if (localeResolver == null) {
                throw new IllegalStateException(
                    "No LocaleResolver found: not in a DispatcherServlet request?");
            }

            // the web context will cache the request locale
            Locale requestLocale = localeResolver.resolveLocale(request);
            try {
                localeResolver.setLocale(request, response, requestLocale);
            }
            catch (IllegalArgumentException ex) {
                if (isIgnoreInvalidLocale()) {
                    logger.debug("Ignoring invalid locale value [" + requestLocale.toString() + "]: " + ex.getMessage());
                }
                else {
                    throw ex;
                }
            }
        } else {
            return super.preHandle(request, response, handler);
        }

        return true;
    }
}
```
**LocaleResolver**的组织结构如下所示：

<!--
```plantuml
interface LocaleResolver
interface LocaleContextResolver
abstract class AbstractLocaleContextResolver

LocaleResolver <|-- LocaleContextResolver
LocaleContextResolver <|.. AbstractLocaleContextResolver
AbstractLocaleContextResolver <|-- SessionLocaleResolver

```
-->

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="391px" preserveAspectRatio="none" style="width:226px;height:391px;" version="1.1" viewBox="0 0 226 391" width="226px" zoomAndPan="magnify"><defs><filter height="300%" id="f1v4eswmbt4855" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><!--class LocaleResolver--><rect fill="#FEFECE" filter="url(#f1v4eswmbt4855)" height="48" id="LocaleResolver" style="stroke: #A80036; stroke-width: 1.5;" width="117" x="52" y="8"></rect><ellipse cx="67" cy="24" fill="#B4A7E5" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M62.9277,19.7651 L62.9277,17.6069 L70.3071,17.6069 L70.3071,19.7651 L67.8418,19.7651 L67.8418,27.8418 L70.3071,27.8418 L70.3071,30 L62.9277,30 L62.9277,27.8418 L65.3931,27.8418 L65.3931,19.7651 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" font-style="italic" lengthAdjust="spacingAndGlyphs" textLength="85" x="81" y="28.5352">LocaleResolver</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="53" x2="168" y1="40" y2="40"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="53" x2="168" y1="48" y2="48"></line><!--class LocaleContextResolver--><rect fill="#FEFECE" filter="url(#f1v4eswmbt4855)" height="48" id="LocaleContextResolver" style="stroke: #A80036; stroke-width: 1.5;" width="161" x="30" y="116"></rect><ellipse cx="45" cy="132" fill="#B4A7E5" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M40.9277,127.7651 L40.9277,125.6069 L48.3071,125.6069 L48.3071,127.7651 L45.8418,127.7651 L45.8418,135.8418 L48.3071,135.8418 L48.3071,138 L40.9277,138 L40.9277,135.8418 L43.3931,135.8418 L43.3931,127.7651 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" font-style="italic" lengthAdjust="spacingAndGlyphs" textLength="129" x="59" y="136.5352">LocaleContextResolver</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="31" x2="190" y1="148" y2="148"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="31" x2="190" y1="156" y2="156"></line><!--class AbstractLocaleContextResolver--><rect fill="#FEFECE" filter="url(#f1v4eswmbt4855)" height="48" id="AbstractLocaleContextResolver" style="stroke: #A80036; stroke-width: 1.5;" width="209" x="6" y="224"></rect><ellipse cx="21" cy="240" fill="#A9DCDF" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M21.1133,235.3481 L19.9595,240.4199 L22.2754,240.4199 Z M19.6191,233.1069 L22.6157,233.1069 L25.9609,245.5 L23.5122,245.5 L22.7485,242.437 L19.4697,242.437 L18.7227,245.5 L16.2739,245.5 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" font-style="italic" lengthAdjust="spacingAndGlyphs" textLength="177" x="35" y="244.5352">AbstractLocaleContextResolver</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="7" x2="214" y1="256" y2="256"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="7" x2="214" y1="264" y2="264"></line><!--class SessionLocaleResolver--><rect fill="#FEFECE" filter="url(#f1v4eswmbt4855)" height="48" id="SessionLocaleResolver" style="stroke: #A80036; stroke-width: 1.5;" width="159" x="31" y="332"></rect><ellipse cx="46" cy="348" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M48.9731,353.6431 Q48.3921,353.9419 47.7529,354.0913 Q47.1138,354.2407 46.4082,354.2407 Q43.9014,354.2407 42.5815,352.5889 Q41.2617,350.937 41.2617,347.8159 Q41.2617,344.6865 42.5815,343.0347 Q43.9014,341.3828 46.4082,341.3828 Q47.1138,341.3828 47.7612,341.5322 Q48.4087,341.6816 48.9731,341.9805 L48.9731,344.7031 Q48.3423,344.1221 47.7488,343.8523 Q47.1553,343.5825 46.5244,343.5825 Q45.1797,343.5825 44.4949,344.6492 Q43.8101,345.7158 43.8101,347.8159 Q43.8101,349.9077 44.4949,350.9744 Q45.1797,352.041 46.5244,352.041 Q47.1553,352.041 47.7488,351.7712 Q48.3423,351.5015 48.9731,350.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="127" x="60" y="352.5352">SessionLocaleResolver</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="32" x2="189" y1="364" y2="364"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="32" x2="189" y1="372" y2="372"></line><!--link LocaleResolver to LocaleContextResolver--><path d="M110.5,76.2911 C110.5,89.8171 110.5,104.1835 110.5,115.8436 " fill="none" id="LocaleResolver-LocaleContextResolver" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="103.5001,76.2373,110.5,56.2373,117.5001,76.2373,103.5001,76.2373" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link LocaleContextResolver to AbstractLocaleContextResolver--><path d="M110.5,184.2911 C110.5,197.8171 110.5,212.1835 110.5,223.8436 " fill="none" id="LocaleContextResolver-AbstractLocaleContextResolver" style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 7.0,7.0;"></path><polygon fill="none" points="103.5001,184.2373,110.5,164.2373,117.5001,184.2373,103.5001,184.2373" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link AbstractLocaleContextResolver to SessionLocaleResolver--><path d="M110.5,292.2911 C110.5,305.8171 110.5,320.1835 110.5,331.8436 " fill="none" id="AbstractLocaleContextResolver-SessionLocaleResolver" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="103.5001,292.2373,110.5,272.2373,117.5001,292.2373,103.5001,292.2373" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--
@startuml
interface LocaleResolver
interface LocaleContextResolver
abstract class AbstractLocaleContextResolver

LocaleResolver <|- - LocaleContextResolver
LocaleContextResolver <|.. AbstractLocaleContextResolver
AbstractLocaleContextResolver <|- - SessionLocaleResolver
@enduml

PlantUML version 1.2018.01(Mon Jan 29 02:08:22 CST 2018)
(GPL source distribution)
Java Runtime: Java(TM) SE Runtime Environment
JVM: Java HotSpot(TM) 64-Bit Server VM
Java Version: 1.8.0_152-b16
Operating System: Mac OS X
OS Version: 10.13.4
Default Encoding: UTF-8
Language: en
Country: CN
--></g></svg>

**AbstractLocaleContextResolver**，该类实现了setLocale方法，在该方法中调用抽象方法setLocaleContext。
```java
@Override
public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
    // LocaleContextResolver
    setLocaleContext(request, response, (locale != null ? new SimpleLocaleContext(locale) : null));
}
```

**SessionLocaleResolver**，实现了抽象方法setLocaleContext，把请求中携带的locale信息存储在session中。
```java
@Override
public void setLocaleContext(HttpServletRequest request, HttpServletResponse response, LocaleContext localeContext) {
    Locale locale = null;
    TimeZone timeZone = null;
    if (localeContext != null) {
        locale = localeContext.getLocale();
        if (localeContext instanceof TimeZoneAwareLocaleContext) {
            timeZone = ((TimeZoneAwareLocaleContext) localeContext).getTimeZone();
        }
    }

    // 存储地区和时区信息
    // this.localeAttributeName默认值为LOCALE_SESSION_ATTRIBUTE_NAME
    // public static final String LOCALE_SESSION_ATTRIBUTE_NAME = SessionLocaleResolver.class.getName() + ".LOCALE";
    WebUtils.setSessionAttribute(request, this.localeAttributeName, locale);
    WebUtils.setSessionAttribute(request, this.timeZoneAttributeName, timeZone);
}
```

以上为Spring MVC获取和存储locale信息的流程，下面我们来看看它是如何根据locale信息处理预先定义的国际化资源。

**MessageSource**定义了根据locale信息获取对应code的字符信息方法，该接口的组织关系如下图所示。在这里以**ResourceBundleMessageSource**为例，分析不带参数并未使用MessageFormat时的处理过程。

<!--
```plantuml
interface MessageSource
interface HierarchicalMessageSource
abstract class AbstractMessageSource
abstract class AbstractResourceBasedMessageSource

MessageSource <|-- HierarchicalMessageSource
HierarchicalMessageSource <|.. AbstractMessageSource
AbstractMessageSource <|-- AbstractResourceBasedMessageSource
AbstractResourceBasedMessageSource <|-- ResourceBundleMessageSource
AbstractResourceBasedMessageSource <|-- ReloadableResourceBundleMessageSource
```
-->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="499px" preserveAspectRatio="none" style="width:541px;height:499px;" version="1.1" viewBox="0 0 541 499" width="541px" zoomAndPan="magnify"><defs><filter height="300%" id="f1vhn93v5gsatw" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><!--class MessageSource--><rect fill="#FEFECE" filter="url(#f1vhn93v5gsatw)" height="48" id="MessageSource" style="stroke: #A80036; stroke-width: 1.5;" width="120" x="192" y="8"></rect><ellipse cx="207" cy="24" fill="#B4A7E5" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M202.9277,19.7651 L202.9277,17.6069 L210.3071,17.6069 L210.3071,19.7651 L207.8418,19.7651 L207.8418,27.8418 L210.3071,27.8418 L210.3071,30 L202.9277,30 L202.9277,27.8418 L205.3931,27.8418 L205.3931,19.7651 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" font-style="italic" lengthAdjust="spacingAndGlyphs" textLength="88" x="221" y="28.5352">MessageSource</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="193" x2="311" y1="40" y2="40"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="193" x2="311" y1="48" y2="48"></line><!--class HierarchicalMessageSource--><rect fill="#FEFECE" filter="url(#f1vhn93v5gsatw)" height="48" id="HierarchicalMessageSource" style="stroke: #A80036; stroke-width: 1.5;" width="188" x="158" y="116"></rect><ellipse cx="173" cy="132" fill="#B4A7E5" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M168.9277,127.7651 L168.9277,125.6069 L176.3071,125.6069 L176.3071,127.7651 L173.8418,127.7651 L173.8418,135.8418 L176.3071,135.8418 L176.3071,138 L168.9277,138 L168.9277,135.8418 L171.3931,135.8418 L171.3931,127.7651 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" font-style="italic" lengthAdjust="spacingAndGlyphs" textLength="156" x="187" y="136.5352">HierarchicalMessageSource</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="159" x2="345" y1="148" y2="148"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="159" x2="345" y1="156" y2="156"></line><!--class AbstractMessageSource--><rect fill="#FEFECE" filter="url(#f1vhn93v5gsatw)" height="48" id="AbstractMessageSource" style="stroke: #A80036; stroke-width: 1.5;" width="168" x="168" y="224"></rect><ellipse cx="183" cy="240" fill="#A9DCDF" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M183.1133,235.3481 L181.9595,240.4199 L184.2754,240.4199 Z M181.6191,233.1069 L184.6157,233.1069 L187.9609,245.5 L185.5122,245.5 L184.7485,242.437 L181.4697,242.437 L180.7227,245.5 L178.2739,245.5 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" font-style="italic" lengthAdjust="spacingAndGlyphs" textLength="136" x="197" y="244.5352">AbstractMessageSource</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="169" x2="335" y1="256" y2="256"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="169" x2="335" y1="264" y2="264"></line><!--class AbstractResourceBasedMessageSource--><rect fill="#FEFECE" filter="url(#f1vhn93v5gsatw)" height="48" id="AbstractResourceBasedMessageSource" style="stroke: #A80036; stroke-width: 1.5;" width="256" x="124" y="332"></rect><ellipse cx="139" cy="348" fill="#A9DCDF" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M139.1133,343.3481 L137.9595,348.4199 L140.2754,348.4199 Z M137.6191,341.1069 L140.6157,341.1069 L143.9609,353.5 L141.5122,353.5 L140.7485,350.437 L137.4697,350.437 L136.7227,353.5 L134.2739,353.5 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" font-style="italic" lengthAdjust="spacingAndGlyphs" textLength="224" x="153" y="352.5352">AbstractResourceBasedMessageSource</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="125" x2="379" y1="364" y2="364"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="125" x2="379" y1="372" y2="372"></line><!--class ResourceBundleMessageSource--><rect fill="#FEFECE" filter="url(#f1vhn93v5gsatw)" height="48" id="ResourceBundleMessageSource" style="stroke: #A80036; stroke-width: 1.5;" width="212" x="6" y="440"></rect><ellipse cx="21" cy="456" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M23.9731,461.6431 Q23.3921,461.9419 22.7529,462.0913 Q22.1138,462.2407 21.4082,462.2407 Q18.9014,462.2407 17.5815,460.5889 Q16.2617,458.937 16.2617,455.8159 Q16.2617,452.6865 17.5815,451.0347 Q18.9014,449.3828 21.4082,449.3828 Q22.1138,449.3828 22.7612,449.5322 Q23.4087,449.6816 23.9731,449.9805 L23.9731,452.7031 Q23.3423,452.1221 22.7488,451.8523 Q22.1553,451.5825 21.5244,451.5825 Q20.1797,451.5825 19.4949,452.6492 Q18.8101,453.7158 18.8101,455.8159 Q18.8101,457.9077 19.4949,458.9744 Q20.1797,460.041 21.5244,460.041 Q22.1553,460.041 22.7488,459.7712 Q23.3423,459.5015 23.9731,458.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="180" x="35" y="460.5352">ResourceBundleMessageSource</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="7" x2="217" y1="472" y2="472"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="7" x2="217" y1="480" y2="480"></line><!--class ReloadableResourceBundleMessageSource--><rect fill="#FEFECE" filter="url(#f1vhn93v5gsatw)" height="48" id="ReloadableResourceBundleMessageSource" style="stroke: #A80036; stroke-width: 1.5;" width="277" x="253.5" y="440"></rect><ellipse cx="268.5" cy="456" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M271.4731,461.6431 Q270.8921,461.9419 270.2529,462.0913 Q269.6138,462.2407 268.9082,462.2407 Q266.4014,462.2407 265.0815,460.5889 Q263.7617,458.937 263.7617,455.8159 Q263.7617,452.6865 265.0815,451.0347 Q266.4014,449.3828 268.9082,449.3828 Q269.6138,449.3828 270.2612,449.5322 Q270.9087,449.6816 271.4731,449.9805 L271.4731,452.7031 Q270.8423,452.1221 270.2488,451.8523 Q269.6553,451.5825 269.0244,451.5825 Q267.6797,451.5825 266.9949,452.6492 Q266.3101,453.7158 266.3101,455.8159 Q266.3101,457.9077 266.9949,458.9744 Q267.6797,460.041 269.0244,460.041 Q269.6553,460.041 270.2488,459.7712 Q270.8423,459.5015 271.4731,458.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="245" x="282.5" y="460.5352">ReloadableResourceBundleMessageSource</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="254.5" x2="529.5" y1="472" y2="472"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="254.5" x2="529.5" y1="480" y2="480"></line><!--link MessageSource to HierarchicalMessageSource--><path d="M252,76.2911 C252,89.8171 252,104.1835 252,115.8436 " fill="none" id="MessageSource-HierarchicalMessageSource" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="245.0001,76.2373,252,56.2373,259.0001,76.2373,245.0001,76.2373" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link HierarchicalMessageSource to AbstractMessageSource--><path d="M252,184.2911 C252,197.8171 252,212.1835 252,223.8436 " fill="none" id="HierarchicalMessageSource-AbstractMessageSource" style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 7.0,7.0;"></path><polygon fill="none" points="245.0001,184.2373,252,164.2373,259.0001,184.2373,245.0001,184.2373" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link AbstractMessageSource to AbstractResourceBasedMessageSource--><path d="M252,292.2911 C252,305.8171 252,320.1835 252,331.8436 " fill="none" id="AbstractMessageSource-AbstractResourceBasedMessageSource" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="245.0001,292.2373,252,272.2373,259.0001,292.2373,245.0001,292.2373" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link AbstractResourceBasedMessageSource to ResourceBundleMessageSource--><path d="M204.4546,392.6779 C184.2737,408.246 161.3078,425.9626 143.3139,439.8436 " fill="none" id="AbstractResourceBasedMessageSource-ResourceBundleMessageSource" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="200.47,386.9109,220.5813,380.2373,209.0212,397.9959,200.47,386.9109" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link AbstractResourceBasedMessageSource to ReloadableResourceBundleMessageSource--><path d="M299.5454,392.6779 C319.7263,408.246 342.6922,425.9626 360.6861,439.8436 " fill="none" id="AbstractResourceBasedMessageSource-ReloadableResourceBundleMessageSource" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="294.9788,397.9959,283.4187,380.2373,303.53,386.9109,294.9788,397.9959" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--
@startuml
interface MessageSource
interface HierarchicalMessageSource
abstract class AbstractMessageSource
abstract class AbstractResourceBasedMessageSource

MessageSource <|- - HierarchicalMessageSource
HierarchicalMessageSource <|.. AbstractMessageSource
AbstractMessageSource <|- - AbstractResourceBasedMessageSource
AbstractResourceBasedMessageSource <|- - ResourceBundleMessageSource
AbstractResourceBasedMessageSource <|- - ReloadableResourceBundleMessageSource
@enduml

PlantUML version 1.2018.01(Mon Jan 29 02:08:22 CST 2018)
(GPL source distribution)
Java Runtime: Java(TM) SE Runtime Environment
JVM: Java HotSpot(TM) 64-Bit Server VM
Java Version: 1.8.0_152-b16
Operating System: Mac OS X
OS Version: 10.13.4
Default Encoding: UTF-8
Language: en
Country: CN
--></g></svg>

**MessageSource**
```java
public interface MessageSource {

	String getMessage(String code, Object[] args, String defaultMessage, Locale locale);

	String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException;

	String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException;

}
```

**AbstractMessageSource**，根据传入的locale和code返回具体的语言信息，定义`protected String resolveCodeWithoutArguments(String code, Locale locale)`，把资源的解析交给子类完成。
```java
@Override
public final String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException {
    // 读取code对应的语言信息
    String msg = getMessageInternal(code, args, locale);
    if (msg != null) {
        return msg;
    }
    String fallback = getDefaultMessage(code);
    if (fallback != null) {
        return fallback;
    }
    throw new NoSuchMessageException(code, locale);
}

protected String getMessageInternal(String code, Object[] args, Locale locale) {
    if (code == null) {
        return null;
    }
    if (locale == null) {
        locale = Locale.getDefault();
    }
    Object[] argsToUse = args;

    if (!isAlwaysUseMessageFormat() && ObjectUtils.isEmpty(args)) {
        // Optimized resolution: no arguments to apply,
        // therefore no MessageFormat needs to be involved.
        // Note that the default implementation still uses MessageFormat;
        // this can be overridden in specific subclasses.
        // 解析资源文件获取语言信息
        String message = resolveCodeWithoutArguments(code, locale);
        if (message != null) {
            return message;
        }
    }

    else {
        // Resolve arguments eagerly, for the case where the message
        // is defined in a parent MessageSource but resolvable arguments
        // are defined in the child MessageSource.
        // 解析带参数的语言信息
        argsToUse = resolveArguments(args, locale);

        MessageFormat messageFormat = resolveCode(code, locale);
        if (messageFormat != null) {
            synchronized (messageFormat) {
                return messageFormat.format(argsToUse);
            }
        }
    }

    // Check locale-independent common messages for the given message code.
    Properties commonMessages = getCommonMessages();
    if (commonMessages != null) {
        String commonMessage = commonMessages.getProperty(code);
        if (commonMessage != null) {
            return formatMessage(commonMessage, args, locale);
        }
    }

    // Not found -> check parent, if any.
    return getMessageFromParent(code, argsToUse, locale);
}
```

**ResourceBundleMessageSource**，重写父类的resolveCodeWithoutArguments方法，只可解析properties资源。
```java
@Override
protected String resolveCodeWithoutArguments(String code, Locale locale) {
    Set<String> basenames = getBasenameSet();
    for (String basename : basenames) {
        // 通过ResourceBundle类读取properties文件
        ResourceBundle bundle = getResourceBundle(basename, locale);
        if (bundle != null) {
            String result = getStringOrNull(bundle, code);
            if (result != null) {
                return result;
            }
        }
    }
    return null;
}
```

除了**ResourceBundleMessageSource**可解析资源信息之外，还有**ReloadableResourceBundleMessageSource**类也可处理，而且支持的资源不止properties文件，还可处理XML文件。

以上就是Spring MVC国际化的主要处理类，如需在业务中做国际化，只需从容器的上下文获取`localeResolver`和`messageSource`两个bean实例即可。

## 2. Hibernate Validator

### 2.1 配置

**spring-mvc.xml**
```xml
<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
	<property name="providerClass" value="org.hibernate.validator.HibernateValidator" />
	<property name="validationMessageSource" ref="messageSource" />
</bean>
```

### 2.2 源码解析

Hibernate Validator默认使用**ResourceBundleMessageInterpolator**进行资源解析。该类实现了**javax.validation**定义的**MessageInterpolator**接口，该类支持EL表达式(${foo})和参数({foo})两种字符串定义方式。

```java
// 具体的解析由以下方法完成
private String interpolateMessage(String message, Context context, Locale locale) throws MessageDescriptorFormatException {
	LocalizedMessage localisedMessage = new LocalizedMessage( message, locale );
	String resolvedMessage = null;

    // 如果有缓存则取出缓存中的存储的资源信息
    if ( cachingEnabled ) {
    	resolvedMessage = resolvedMessages.get( localisedMessage );
    }

    // 未缓存，则从资源文件获取，通过以下三步完成
    if ( resolvedMessage == null ) {
        // 用户定义的资源
    	ResourceBundle userResourceBundle = userResourceBundleLocator
            .getResourceBundle( locale );
        //  hibernate validator 内置资源
    	ResourceBundle defaultResourceBundle = defaultResourceBundleLocator
            .getResourceBundle( locale );

    	String userBundleResolvedMessage;
    	resolvedMessage = message;
    	boolean evaluatedDefaultBundleOnce = false;
    	do {
            // 1. 在userResourceBundle中搜索resolvedMessage对应的字符串(递归)
            userBundleResolvedMessage = interpolateBundleMessage(
                resolvedMessage, userResourceBundle, locale, true
            );

            // 3. 检测中断循环条件
            // 3.1 至少执行一次
            // 3.2 userBundleResolvedMessage和resolvedMessage相同
            if ( evaluatedDefaultBundleOnce &&
                    !hasReplacementTakenPlace( userBundleResolvedMessage, resolvedMessage ) ) {
            	break;
            }

            // 2. 在defaultResourceBundle中搜索userBundleResolvedMessage对应的字符串(非递归)
            resolvedMessage = interpolateBundleMessage(
                userBundleResolvedMessage,
                defaultResourceBundle,
                locale,
                false
            );
            evaluatedDefaultBundleOnce = true;
    	} while ( true );
    }

    // 缓存处理后的资源信息
    if ( cachingEnabled ) {
    	String cachedResolvedMessage = resolvedMessages.putIfAbsent( localisedMessage, resolvedMessage );
    	if ( cachedResolvedMessage != null ) {
    		resolvedMessage = cachedResolvedMessage;
    	}
    }

    // 根据缓存处理资源信息
    // 4. 解析 参数表达式（内置资源）
    List<Token> tokens = null;
    if ( cachingEnabled ) {
    	tokens = tokenizedParameterMessages.get( resolvedMessage );
    }
    if ( tokens == null ) {
    	TokenCollector tokenCollector = new TokenCollector( resolvedMessage, InterpolationTermType.PARAMETER );
    	tokens = tokenCollector.getTokenList();

    	if ( cachingEnabled ) {
    		tokenizedParameterMessages.putIfAbsent( resolvedMessage, tokens );
    	}
    }
    resolvedMessage = interpolateExpression(
    		new TokenIterator( tokens ),
    		context,
    		locale
    );

    // 5. 解析 EL表达式（内置资源）
    tokens = null;
    if ( cachingEnabled ) {
    	tokens = tokenizedELMessages.get( resolvedMessage );
    }
    if ( tokens == null ) {
    	TokenCollector tokenCollector = new TokenCollector( resolvedMessage, InterpolationTermType.EL );
    	tokens = tokenCollector.getTokenList();

    	if ( cachingEnabled ) {
    		tokenizedELMessages.putIfAbsent( resolvedMessage, tokens );
    	}
    }
    resolvedMessage = interpolateExpression(
    		new TokenIterator( tokens ),
    		context,
    		locale
    );

    // 检测带转义符的字符串,替换为原始字符串
    resolvedMessage = replaceEscapedLiterals( resolvedMessage );

    return resolvedMessage;
}

...

// 获取message对应的value并处理表达式
private String interpolateBundleMessage(String message, ResourceBundle bundle, Locale locale, boolean recursive)
        throws MessageDescriptorFormatException {
    TokenCollector tokenCollector = new TokenCollector( message, InterpolationTermType.PARAMETER );
    TokenIterator tokenIterator = new TokenIterator( tokenCollector.getTokenList() );
    while ( tokenIterator.hasMoreInterpolationTerms() ) {
        String term = tokenIterator.nextInterpolationTerm();

        // 根据表达式中的key获取对应的值
        String resolvedParameterValue = resolveParameter(
                term, bundle, locale, recursive
        );

        // 用resolvedParameterValue的值替换key
        tokenIterator.replaceCurrentInterpolationTerm( resolvedParameterValue );
    }

    // 返回处理之后的字符
    return tokenIterator.getInterpolatedMessage();
}

// 处理内置资源中的表达式，如注解和表达式的值绑定处理等
private String interpolateExpression(TokenIterator tokenIterator, Context context, Locale locale)
        throws MessageDescriptorFormatException {
    while ( tokenIterator.hasMoreInterpolationTerms() ) {
        String term = tokenIterator.nextInterpolationTerm();

        InterpolationTerm expression = new InterpolationTerm( term, locale );
        String resolvedExpression = expression.interpolate( context );
        tokenIterator.replaceCurrentInterpolationTerm( resolvedExpression );
    }
    return tokenIterator.getInterpolatedMessage();
}

// 以parameterName为key获取对应的value，如果recursive为true，则调用interpolateBundleMessage循环往复,直到value中的表达式全部解析
private String resolveParameter(String parameterName, ResourceBundle bundle, Locale locale, boolean recursive)
        throws MessageDescriptorFormatException {
    String parameterValue;
    try {
        if ( bundle != null ) {
            parameterValue = bundle.getString( removeCurlyBraces( parameterName ) );
            if ( recursive ) {
                parameterValue = interpolateBundleMessage( parameterValue, bundle, locale, recursive );
            }
        }
        else {
            parameterValue = parameterName;
        }
    }
    catch ( MissingResourceException e ) {
        // 返回传入的key
        parameterValue = parameterName;
    }
    return parameterValue;
}

// 移除"{"和"}"字符
private String removeCurlyBraces(String parameter) {
    return parameter.substring( 1, parameter.length() - 1 );
}
```
