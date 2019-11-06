---
layout: post
category : Tech
title : Tomcat源码阅读笔记 - Connector 详解
tags : [java, tomcat, sourcecode]
excerpt: Tomcat源码阅读笔记总结，记录在阅读过程中的要点和相关内容分析
---
{% include JB/setup %}

环境:

* Tomcat 9.0.17

`org.apache.catalina.connector.Connector` 定义了 Tomcat 运行中的相关配置项，包括支持协议，端口，以及解析相关参数等，实例由`org.apache.catalinaService`持有和管理，下面我们来逐一看看它的这些属性。

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
protected HashSet<String> parseBodyMethodsSet;
protected boolean useIPVHosts = false;
protected final String protocolHandlerClassName;
protected final ProtocolHandler protocolHandler;
protected Adapter adapter = null;
private Charset uriCharset = StandardCharsets.UTF_8;
protected boolean useBodyEncodingForURI = false;
```

它的属性有以上这些(无关本文的属性已去除)，其中的部分属性可参考该包下面的`mbeans-descriptors.xml`文件，该文件包括 `Connector` 的可配置属性和 `ProtocolHandler` 实例的共用属性，可通过 `server.xml` 中的 `connector` 标签进行配置。

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

一个请求创建 `cookie` 的最大数量，默认为200，-1为不限制

### maxParameterCount

`GET` 和 `POST` 请求中参数的最大数量，默认为10000。小于0为不限制

### maxPostSize

`POST` 请求数据的最大长度，默认为2MB

### protocol

支持协议:

##### 1. HTTP/1.1

* org.apache.coyote.http11.Http11AprProtocol
* org.apache.coyote.http11.Http11NioProtocol
* org.apache.coyote.http11.Http11Nio2Protocol

##### 2. AJP/1.3

* org.apache.coyote.ajp.AjpAprProtocol
* org.apache.coyote.ajp.AjpNioProtocol
* org.apache.coyote.ajp.AjpNio2Protocol
