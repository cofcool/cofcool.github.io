---
layout: post
category : Tech
title : Tomcat源码阅读笔记 - 详解 Connector
tags : [java, tomcat, sourcecode]
excerpt: Tomcat源码阅读笔记总结，记录在阅读过程中的要点和相关内容分析，本文对 Connector 进行解析
---
{% include JB/setup %}

环境:

* Tomcat 9.0.17

`org.apache.catalina.connector.Connector` 定义了 Tomcat 运行中的相关配置项，包括支持协议，端口，以及解析相关参数等，实例由`org.apache.catalina.Service`持有和管理，下面我们来逐一看看它的这些属性。

```java
protected boolean allowTrace = false;
protected long asyncTimeout = 30000;
protected boolean enableLookups = false;
protected boolean xpoweredBy = false;
protected String proxyName = null;
protected int proxyPort = 0;
protected int redirectPort = 443;
protected String scheme = "http";
protected boolean secure = false;
private int maxCookieCount = 200;
protected int maxParameterCount = 10000;
protected int maxPostSize = 2 * 1024 * 1024;
protected int maxSavePostSize = 4 * 1024;
protected String parseBodyMethods = "POST";
protected boolean useIPVHosts = false;
protected final String protocolHandlerClassName;
protected final ProtocolHandler protocolHandler;
private Charset uriCharset = StandardCharsets.UTF_8;
protected boolean useBodyEncodingForURI = false;
```

它的属性有以上这些(无关本文的属性已去除)，其中的部分属性可参考该包下面的`mbeans-descriptors.xml`文件，该文件包括 `Connector` 的可配置属性和 `ProtocolHandler` 的属性，可通过 `server.xml` 中的 `connector` 标签进行配置，并由`ConnectorMBean`把配置项注入到`Connector`中，具体过程可参考`org.apache.tomcat.util.modeler.Registry`和`org.apache.catalina.mbeans.MBeanUtils`。


### allowTrace

是否支持 [TRACE](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/TRACE) 请求

### asyncTimeout

异步请求超时时间，默认 30s

### enableLookups

是否寻找远程主机名

### xpoweredBy

响应头是否添加 `X-Powered-By` 字段

### proxyName

代理服务器名称，影响 `request.serverName`

### proxyPort

代理服务器端口，影响 `request.serverPort`

### redirectPort

从非 `SSL` 定向到 `SSL` 的端口

### scheme

默认请求为 `http`

### secure

是否使用 `https`

### maxCookieCount

每个请求创建 `cookie` 的最大数量，默认为200，-1为不限制

### maxParameterCount

`GET` 和 `POST` 请求中参数的最大数量，默认为10000。小于0为不限制

### maxPostSize

`POST` 请求数据的最大长度，默认为2MB，会影响文件上传，参数解析，具体可参考`org.apache.catalina.connector.Request`，以`parseParameters`方法为例:

```java
int maxPostSize = connector.getMaxPostSize();
// 比较长度
if ((maxPostSize >= 0) && (len > maxPostSize)) {
    Context context = getContext();
    if (context != null && context.getLogger().isDebugEnabled()) {
        context.getLogger().debug(
                sm.getString("coyoteRequest.postTooLarge"));
    }
    checkSwallowInput();
    // 解析失败，设置失败原因
    parameters.setParseFailedReason(FailReason.POST_TOO_LARGE);
    return;
}
```

### maxSavePostSize

授权时允许存储到容器中数据的最大长度，该属性只会影响到`FormAuthenticator`

### parseBodyMethods

当请求的`Content-Type`为`application/x-www-form-urlencoded`，并且请求方式在`parseBodyMethods`中定义的请求会进行参数解析，默认为`POST`，参考`org.apache.catalina.connector.Request.parseParameters`方法

### useIPVHosts

是否开启虚拟主机

### useBodyEncodingForURI

请求"URI"的编码是否和请求体一致，如果是，则使用请求体的编码来解析该`QueryString`，如果否，则使用"utf-8"

```java
// org.apache.catalina.connector.Request
// 解析编码
private Charset getCharset() {
    Charset charset = null;
    try {
        // 从请求中读取
        charset = coyoteRequest.getCharset();
    } catch (UnsupportedEncodingException e) {
        // Ignore
    }
    if (charset != null) {
        return charset;
    }
    Context context = getContext();
    if (context != null) {
        // 从配置中读取
        String encoding = context.getRequestCharacterEncoding();
        if (encoding != null) {
            try {
                return B2CConverter.getCharset(encoding);
            } catch (UnsupportedEncodingException e) {
                // Ignore
            }
        }
    }

    // ISO_8859_1
    return org.apache.coyote.Constants.DEFAULT_BODY_CHARSET;
}
```

### uriCharset

"URI"使用的编码，默认为"utf-8"，可调用`setURIEncoding`方法可进行配置

### protocol

支持协议:

##### 1. HTTP/1.1

* org.apache.coyote.http11.Http11AprProtocol
* org.apache.coyote.http11.Http11NioProtocol
* org.apache.coyote.http11.Http11Nio2Protocol

如果需要"HTTP2"，配置`protocol`下的`UpgradeProtocol`即可,"SSL"也是同理，添加`SSLHostConfig`即可

##### 3. AJP/1.3

* org.apache.coyote.ajp.AjpAprProtocol
* org.apache.coyote.ajp.AjpNioProtocol
* org.apache.coyote.ajp.AjpNio2Protocol

解析过程:

```java
// 解析 protocol
public Connector(String protocol) {
    boolean aprConnector = AprLifecycleListener.isAprAvailable() &&
            AprLifecycleListener.getUseAprConnector();
    if ("HTTP/1.1".equals(protocol) || protocol == null) {
        if (aprConnector) {
            protocolHandlerClassName = "org.apache.coyote.http11.Http11AprProtocol";
        } else {
            protocolHandlerClassName = "org.apache.coyote.http11.Http11NioProtocol";
        }
    } else if ("AJP/1.3".equals(protocol)) {
        if (aprConnector) {
            protocolHandlerClassName = "org.apache.coyote.ajp.AjpAprProtocol";
        } else {
            protocolHandlerClassName = "org.apache.coyote.ajp.AjpNioProtocol";
        }
    } else {
        protocolHandlerClassName = protocol;
    }
    // Instantiate protocol handler
    ProtocolHandler p = null;
    try {
        Class<?> clazz = Class.forName(protocolHandlerClassName);
        p = (ProtocolHandler) clazz.getConstructor().newInstance();
    } catch (Exception e) {
        log.error(sm.getString(
                "coyoteConnector.protocolHandlerInstantiationFailed"), e);
    } finally {
        this.protocolHandler = p;
    }
    // Default for Connector depends on this system property
    setThrowOnFailure(Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE"));
}
```

`ProtocolHandler`相当的重要，"HTTP"等协议的解析全靠它来完成，可以说是"Tomcat"的核心组件，所以它的代码设计也会很复杂，因此会在以后专门写一篇文章来解析。