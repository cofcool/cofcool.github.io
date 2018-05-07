---
layout: post
category : Tech
title : Spring MVC + Shiro 原理解析
tags : [java, shiro]
excerpt: 在Spring MVC中，`DelegatingFilterProxy`会代理Spring上下文中实现`Servlet Filter`接口的类，该类负责控制调用项目中的`Servlet Filter`类，包括`ShiroFilter`。下面我们来看看具体流程和实现。
---
{% include JB/setup %}

在Spring MVC中，`DelegatingFilterProxy`会代理Spring上下文中实现`Servlet Filter`接口的类，该类负责控制调用项目中的`Servlet Filter`类，包括`ShiroFilter`。下面我们来看看具体流程和实现。

目录

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. DelegatingFilterProxy](#1-delegatingfilterproxy)
* [2. Shiro处理流程](#2-shiro处理流程)

<!-- /code_chunk_output -->


## 1. DelegatingFilterProxy

DelegatingFilterProxy会在上下文中寻找名为`targetBeanName(web.xml中配置)`的值的bean，并代理该实例。

**DelegatingFilterProxy**
```java
// 创建delegate
protected Filter initDelegate(WebApplicationContext wac) throws ServletException {
	Filter delegate = wac.getBean(getTargetBeanName(), Filter.class);
	if (isTargetFilterLifecycle()) {
		delegate.init(getFilterConfig());
	}
	return delegate;
}
```

流程图:
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="327px" preserveAspectRatio="none" style="width:698px;height:327px;" version="1.1" viewBox="0 0 698 327" width="698px" zoomAndPan="magnify"><defs><filter height="300%" id="fk13qptkiwc0h" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="31" x2="31" y1="38.4883" y2="287.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="98" x2="98" y1="38.4883" y2="287.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="181" x2="181" y1="38.4883" y2="287.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="312" x2="312" y1="38.4883" y2="287.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="491" x2="491" y1="38.4883" y2="287.041"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="631" x2="631" y1="38.4883" y2="287.041"></line><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="42" x="8" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="28" x="15" y="23.5352">请求</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="42" x="8" y="286.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="28" x="15" y="306.5762">请求</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="65" x="64" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="51" x="71" y="23.5352">Tomcat</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="65" x="64" y="286.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="51" x="71" y="306.5762">Tomcat</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="72" x="143" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="58" x="150" y="23.5352">web.xml</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="72" x="143" y="286.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="58" x="150" y="306.5762">web.xml</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="162" x="229" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="148" x="236" y="23.5352">DelegatingFilterProxy</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="162" x="229" y="286.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="148" x="236" y="306.5762">DelegatingFilterProxy</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="168" x="405" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="154" x="412" y="23.5352">ShiroFilterFactoryBean</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="168" x="405" y="286.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="154" x="412" y="306.5762">ShiroFilterFactoryBean</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="85" x="587" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="71" x="594" y="23.5352">ShiroFilter</text><rect fill="#FEFECE" filter="url(#fk13qptkiwc0h)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="85" x="587" y="286.041"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="71" x="594" y="306.5762">ShiroFilter</text><polygon fill="#A80036" points="86.5,50.4883,96.5,54.4883,86.5,58.4883,90.5,54.4883" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="31" x2="92.5" y1="54.4883" y2="54.4883"></line><polygon fill="#A80036" points="169,64.4883,179,68.4883,169,72.4883,173,68.4883" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="98.5" x2="175" y1="68.4883" y2="68.4883"></line><polygon fill="#A80036" points="300,78.4883,310,82.4883,300,86.4883,304,82.4883" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="181" x2="306" y1="82.4883" y2="82.4883"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="312" x2="354" y1="111.7988" y2="111.7988"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="354" x2="354" y1="111.7988" y2="124.7988"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="313" x2="354" y1="124.7988" y2="124.7988"></line><polygon fill="#A80036" points="323,120.7988,313,124.7988,323,128.7988,319,124.7988" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="48" x="319" y="107.0566">doFilter</text><polygon fill="#A80036" points="479,149.7988,489,153.7988,479,157.7988,483,153.7988" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="312" x2="485" y1="153.7988" y2="153.7988"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="76" x="319" y="149.3672">initDelegate</text><polygon fill="#A80036" points="619.5,179.1094,629.5,183.1094,619.5,187.1094,623.5,183.1094" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="491" x2="625.5" y1="183.1094" y2="183.1094"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="61" x="498" y="178.6777">getObject</text><polygon fill="#A80036" points="323,193.4199,313,197.4199,323,201.4199,319,197.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="317" x2="630.5" y1="197.4199" y2="197.4199"></line><polygon fill="#A80036" points="619.5,222.4199,629.5,226.4199,619.5,230.4199,623.5,226.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="312" x2="625.5" y1="226.4199" y2="226.4199"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="97" x="319" y="221.9883">invokeDelegate</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="631.5" x2="673.5" y1="256.041" y2="256.041"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="673.5" x2="673.5" y1="256.041" y2="269.041"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="632.5" x2="673.5" y1="269.041" y2="269.041"></line><polygon fill="#A80036" points="642.5,265.041,632.5,269.041,642.5,273.041,638.5,269.041" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="48" x="638.5" y="251.2988">doFilter</text><!--
@startuml
请求 -> Tomcat
Tomcat -> web.xml
web.xml -> DelegatingFilterProxy
DelegatingFilterProxy -> DelegatingFilterProxy: doFilter
DelegatingFilterProxy -> ShiroFilterFactoryBean: initDelegate
ShiroFilterFactoryBean -> ShiroFilter: getObject
ShiroFilter -> DelegatingFilterProxy
DelegatingFilterProxy -> ShiroFilter: invokeDelegate
ShiroFilter -> ShiroFilter: doFilter
@enduml

PlantUML version 1.2018.01(Mon Jan 29 02:08:22 CST 2018)
(GPL source distribution)
Java Runtime: Java(TM) SE Runtime Environment
JVM: Java HotSpot(TM) 64-Bit Server VM
Java Version: 10.0.1+10
Operating System: Mac OS X
OS Version: 10.13.4
Default Encoding: UTF-8
Language: en
Country: CN
--></g></svg>
<!--
```plantuml
请求 -> Tomcat
Tomcat -> web.xml
web.xml -> DelegatingFilterProxy
DelegatingFilterProxy -> DelegatingFilterProxy: doFilter
DelegatingFilterProxy -> ShiroFilterFactoryBean: initDelegate
ShiroFilterFactoryBean -> ShiroFilter: getObject
ShiroFilter -> DelegatingFilterProxy
DelegatingFilterProxy -> ShiroFilter: invokeDelegate
ShiroFilter -> ShiroFilter: doFilter
```
-->

## 2. Shiro处理流程

`ShiroFilterFactoryBean`，负责创建和维护ShiroFilter实例，实现了Spring的BeanPostProcessor和FactoryBean，通过`postProcessBeforeInitialization`方法来缓存上下文中存在的filter，另外管理以下配置：
* securityManager：安全管理器
* loginUrl：登陆路径
* successUrl：登陆成功跳转路径
* unauthorizedUrl：未授权路径
* filterChainDefinitions：过滤器和路径规则映射关系
* ...

**ShiroFilterFactoryBean**
```java
// 创建FilterChainManager实例
protected FilterChainManager createFilterChainManager() {

    DefaultFilterChainManager manager = new DefaultFilterChainManager();
    Map<String, Filter> defaultFilters = manager.getFilters();

    // 配置全局默认提供的filter
    // org.apache.shiro.web.filter.mgt.DefaultFilter,定义了默认实现的filter
    //
    // anon(AnonymousFilter.class),
    // authc(FormAuthenticationFilter.class),
    // authcBasic(BasicHttpAuthenticationFilter.class),
    // logout(LogoutFilter.class),
    // noSessionCreation(NoSessionCreationFilter.class),
    // perms(PermissionsAuthorizationFilter.class),
    // port(PortFilter.class),
    // rest(HttpMethodPermissionFilter.class),
    // roles(RolesAuthorizationFilter.class),
    // ssl(SslFilter.class),
    // user(UserFilter.class);
    for (Filter filter : defaultFilters.values()) {
        applyGlobalPropertiesIfNecessary(filter);
    }

    // 配置filters属性中定义的filter
    Map<String, Filter> filters = getFilters();
    if (!CollectionUtils.isEmpty(filters)) {
        for (Map.Entry<String, Filter> entry : filters.entrySet()) {
            String name = entry.getKey();
            Filter filter = entry.getValue();
            applyGlobalPropertiesIfNecessary(filter);
            if (filter instanceof Nameable) {
                ((Nameable) filter).setName(name);
            }

            manager.addFilter(name, filter, false);
        }
    }

    // 创建filter链，需定义filterChainDefinitions
    // 解析filterChainDefinitions，确立路由规则和定义的filter别名关系，并创建filter链
    Map<String, String> chains = getFilterChainDefinitionMap();
    if (!CollectionUtils.isEmpty(chains)) {
        for (Map.Entry<String, String> entry : chains.entrySet()) {
            String url = entry.getKey();
            String chainDefinition = entry.getValue();
            manager.createChain(url, chainDefinition);
        }
    }

    return manager;
}
/**
 *  创建AbstractShiroFilter实例，该类实现了Filter接口
 *  确保securityManager为WebSecurityManager
 *  根据缓存的filters创建FilterChainManager和PathMatchingFilterChainResolver
 */
protected AbstractShiroFilter createInstance() throws Exception {
    // 检查参数
    log.debug("Creating Shiro Filter instance.");

    SecurityManager securityManager = getSecurityManager();
    if (securityManager == null) {
        String msg = "SecurityManager property must be set.";
        throw new BeanInitializationException(msg);
    }

    if (!(securityManager instanceof WebSecurityManager)) {
        String msg = "The security manager does not implement the WebSecurityManager interface.";
        throw new BeanInitializationException(msg);
    }

    // 管理路由规则和filter的映射关系
    FilterChainManager manager = createFilterChainManager();

    // 匹配路由规则（Ant-style），并调用FilterChainManager做相应的操作
    PathMatchingFilterChainResolver chainResolver = new PathMatchingFilterChainResolver();
    chainResolver.setFilterChainManager(manager);

    // SpringShiroFilter包装securityManager，chainResolver，便于调用
    return new SpringShiroFilter((WebSecurityManager) securityManager, chainResolver);
}
```

Shiro执行过程:
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="342px" preserveAspectRatio="none" style="width:695px;height:342px;" version="1.1" viewBox="0 0 695 342" width="695px" zoomAndPan="magnify"><defs><filter height="300%" id="felgahdv4qkxf" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="52" x2="52" y1="38.4883" y2="302.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="141" x2="141" y1="38.4883" y2="302.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="276.5" x2="276.5" y1="38.4883" y2="302.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="456" x2="456" y1="38.4883" y2="302.3516"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="612" x2="612" y1="38.4883" y2="302.3516"></line><rect fill="#FEFECE" filter="url(#felgahdv4qkxf)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="85" x="8" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="71" x="15" y="23.5352">ShiroFilter</text><rect fill="#FEFECE" filter="url(#felgahdv4qkxf)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="85" x="8" y="301.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="71" x="15" y="321.8867">ShiroFilter</text><rect fill="#FEFECE" filter="url(#felgahdv4qkxf)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="64" x="107" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="50" x="114" y="23.5352">Subject</text><rect fill="#FEFECE" filter="url(#felgahdv4qkxf)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="64" x="107" y="301.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="50" x="114" y="321.8867">Subject</text><rect fill="#FEFECE" filter="url(#felgahdv4qkxf)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="157" x="196.5" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="143" x="203.5" y="23.5352">WebSecurityManager</text><rect fill="#FEFECE" filter="url(#felgahdv4qkxf)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="157" x="196.5" y="301.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="143" x="203.5" y="321.8867">WebSecurityManager</text><rect fill="#FEFECE" filter="url(#felgahdv4qkxf)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="138" x="385" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="124" x="392" y="23.5352">AuthorizingRealm</text><rect fill="#FEFECE" filter="url(#felgahdv4qkxf)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="138" x="385" y="301.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="124" x="392" y="321.8867">AuthorizingRealm</text><rect fill="#FEFECE" filter="url(#felgahdv4qkxf)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="147" x="537" y="3"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="133" x="544" y="23.5352">CredentialsMatcher</text><rect fill="#FEFECE" filter="url(#felgahdv4qkxf)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="147" x="537" y="301.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="133" x="544" y="321.8867">CredentialsMatcher</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="52.5" x2="94.5" y1="69.7988" y2="69.7988"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="94.5" x2="94.5" y1="69.7988" y2="82.7988"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="53.5" x2="94.5" y1="82.7988" y2="82.7988"></line><polygon fill="#A80036" points="63.5,78.7988,53.5,82.7988,63.5,86.7988,59.5,82.7988" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="47" x="59.5" y="65.0566">logined</text><polygon fill="#A80036" points="129,107.7988,139,111.7988,129,115.7988,133,111.7988" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="52.5" x2="135" y1="111.7988" y2="111.7988"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="32" x="59.5" y="107.3672">login</text><polygon fill="#A80036" points="265,137.1094,275,141.1094,265,145.1094,269,141.1094" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="141" x2="271" y1="141.1094" y2="141.1094"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="32" x="148" y="136.6777">login</text><polygon fill="#A80036" points="444,166.4199,454,170.4199,444,174.4199,448,170.4199" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="277" x2="450" y1="170.4199" y2="170.4199"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="155" x="284" y="165.9883">doGetAuthenticationInfo</text><polygon fill="#A80036" points="288,180.7305,278,184.7305,288,188.7305,284,184.7305" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="282" x2="455" y1="184.7305" y2="184.7305"></line><polygon fill="#A80036" points="600.5,209.7305,610.5,213.7305,600.5,217.7305,604.5,213.7305" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="277" x2="606.5" y1="213.7305" y2="213.7305"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="125" x="284" y="209.2988">doCredentialsMatch</text><polygon fill="#A80036" points="288,224.041,278,228.041,288,232.041,284,228.041" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="282" x2="611.5" y1="228.041" y2="228.041"></line><polygon fill="#A80036" points="152,238.041,142,242.041,152,246.041,148,242.041" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="146" x2="276" y1="242.041" y2="242.041"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="141" x2="183" y1="271.3516" y2="271.3516"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="183" x2="183" y1="271.3516" y2="284.3516"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="142" x2="183" y1="284.3516" y2="284.3516"></line><polygon fill="#A80036" points="152,280.3516,142,284.3516,152,288.3516,148,284.3516" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="122" x="148" y="266.6094">store user message</text><!--
@startuml
ShiroFilter -> ShiroFilter:logined
ShiroFilter -> Subject: login
Subject -> WebSecurityManager: login
WebSecurityManager -> AuthorizingRealm: doGetAuthenticationInfo
AuthorizingRealm -> WebSecurityManager
WebSecurityManager -> CredentialsMatcher: doCredentialsMatch
CredentialsMatcher -> WebSecurityManager
WebSecurityManager -> Subject
Subject -> Subject: store user message
@enduml

PlantUML version 1.2018.01(Mon Jan 29 02:08:22 CST 2018)
(GPL source distribution)
Java Runtime: Java(TM) SE Runtime Environment
JVM: Java HotSpot(TM) 64-Bit Server VM
Java Version: 10.0.1+10
Operating System: Mac OS X
OS Version: 10.13.4
Default Encoding: UTF-8
Language: en
Country: CN
--></g></svg>

<!--
```plantuml
ShiroFilter -> ShiroFilter:logined
ShiroFilter -> Subject: login
Subject -> WebSecurityManager: login
WebSecurityManager -> AuthorizingRealm: doGetAuthenticationInfo
AuthorizingRealm -> WebSecurityManager
WebSecurityManager -> CredentialsMatcher: doCredentialsMatch
CredentialsMatcher -> WebSecurityManager
WebSecurityManager -> Subject
Subject -> Subject: store user message
```
-->

有关登陆过程的详细代码分析可阅读之前的文章 [Shiro 登陆原理解析以及配置多个Realm](https://cofcool.github.io/tech/2018/03/20/shiro-multi-realms)了解更多。

通过以上分析我们可得知，filter和路由规则是实现权限管理等功能的必要条件，Shiro通过一系列的路由匹配，找到合适的Filter，从而执行对应的拦截逻辑处理。如果默认（DefaultFilter）定义的规则不满足于需求，可添加自定义filter，并配置对应的路由规则即可。

现在看来Shiro是不是很简单而且易用，👍。
