---
layout: post
category : Tech
title : Spring MVC + Shiro 原理解析
tags : [java, shiro]
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
````

Shiro执行过程:
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
有关登陆过程的详细代码分析可阅读之前的文章 [Shiro 登陆原理解析以及配置多个Realm](https://cofcool.github.io/tech/2018/03/20/shiro-multi-realms)了解更多。

通过以上分析我们可得知，filter和路由规则是实现权限管理等功能的必要条件，Shiro通过一系列的路由匹配，找到合适的Filter，从而执行对应的拦截逻辑处理。如果默认（DefaultFilter）定义的规则不满足于需求，可添加自定义filter，并配置对应的路由规则即可。

现在看来Shiro是不是很简单而且易用，👍。
