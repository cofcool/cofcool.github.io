# 使用Spring MVC + Shiro 原理解析

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

protected AbstractShiroFilter createInstance() throws Exception {

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

    FilterChainManager manager = createFilterChainManager();

    //Expose the constructed FilterChainManager by first wrapping it in a
    // FilterChainResolver implementation. The AbstractShiroFilter implementations
    // do not know about FilterChainManagers - only resolvers:
    PathMatchingFilterChainResolver chainResolver = new PathMatchingFilterChainResolver();
    chainResolver.setFilterChainManager(manager);

    //Now create a concrete ShiroFilter instance and apply the acquired SecurityManager and built
    //FilterChainResolver.  It doesn't matter that the instance is an anonymous inner class
    //here - we're just using it because it is a concrete AbstractShiroFilter instance that accepts
    //injection of the SecurityManager and FilterChainResolver:
    return new SpringShiroFilter((WebSecurityManager) securityManager, chainResolver);
}
````

```plantuml
ShiroFilter -> ShiroFilter:logined
ShiroFilter -> Subject: do not login
Subject -> WebSecurityManager: login
WebSecurityManager -> AuthorizingRealm: doGetAuthenticationInfo
AuthorizingRealm -> WebSecurityManager
WebSecurityManager -> CredentialsMatcher: doCredentialsMatch
CredentialsMatcher -> WebSecurityManager
WebSecurityManager -> Subject
Subject -> Subject: store user message
```
