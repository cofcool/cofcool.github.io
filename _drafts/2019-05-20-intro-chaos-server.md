---
layout: post
category: Tech
title: 简单介绍 Chaos Server
tags: [java]
excerpt: chaos-server
---

{% include JB/setup %}


<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [ 1. 模块说明](#1-模块说明)
- [ 2. 配置 ](#2-配置)
- [ 3. 使用](#3-使用)
  - [ 3.1 授权处理](#31-授权处理)
  - [ 3.2 Service 层](#32-service-层)
  - [ 3.3 Dao 层](#33-dao-层)
  - [ 3.4 Controller 层](#34-controller-层)
    - [ 3.4.1 Json解析](#341-json解析)
  - [ 3.5 异常处理](#35-异常处理)
    - [ 3.5.1 默认处理异常类型](#351-默认处理异常类型)
    - [ 3.5.2 描述信息](#352-描述信息)

<!-- /code_chunk_output -->

基于 Spring Boot 的 Java Web Server 框架, 简化开发, 封装了常见的企业级项目开发框架, 如 `Mybaits`, `Spring Security` 等。

### 1. 模块说明

项目模块:

1. common
2. core
3. data-jpa
4. data-mybatis
5. data-redis
6. security-shiro
7. security-spring
8. actuator
9. component-processor
10. boot-starter

第三方依赖:

1. Spring Boot
2. Spring data Jpa
3. MyBatis
4. Mybatis-PageHelper
5. Jackson
7. Shiro
8. Spring Security
9. Spring data Redis
10. Ehcache
10. ...


### 2. 配置 


```properties
# 项目运行模式
chaos.development.mode=dev
# 项目版本
chaos.development.version=100
# 是否开启验证码
chaos.auth.using-captcha=false
# 定义扫描 Scanned 注解的路径
chaos.development.annotation-path=net.cofcool.chaos.server.demo
# shiro授权路径配置
chaos.auth.urls=/auth/**\=anon\n/error\=anon\n/**\=authc
# 登陆路径
chaos.auth.login-url=/auth/login
# 注入数据key配置, 多个时以","分隔
chaos.auth.checked-keys=id
```

### 3. 使用

业务相关Service可继承`DataAccess`接口, 实现类可继承`SimpleService`抽象类。


使用`Page`类封装分页的相关数据, ORM模块的`Paging`继承并扩展。

* Mybatis: 通过`PageHelper`分页。
* Jpa: 通过`Pageable`分页。

**异常**处理时, 自定义业务相关异常需继承`ServiceException`。如需设定异常级别, 实现`ExceptionLevel`接口即可, 该级别影响异常的打印。`ServiceException`已实现该接口, 默认为最高级别。

#### 3.1 授权处理

封装了`Apache Shiro`和`Spring Security`, 应用可依赖`security-shiro`模块或`security-spring`模块来实现授权管理。

登录:

![image]({{ site.url }}/public/upload/images/auth_login.svg)

`AuthService`类定义了登录等操作, 应用不需要实现该类, 只需引用该组件即可, `Shiro`模块需要通过调用该类的`login`来实现登录，`Spring Security`的登录由"filter"完成，因此不需要调用该方法。

`UserAuthorizationService`定义应用操作, 应用需实现该类。

`PasswordProcessor`定义密码处理操作, 应用需实现该类。

`User`存储用户信息。

#### 3.2 Service 层

`DataAccess`定义"Service"常用的方法。

`Result`封装运行结果，提供两个之接口，`QueryResult`封装查询结果，其它情况由`ExecuteResult`处理。

抽象类`SimpleService`实现`DataAccess`类，实现`query`方法，简化分页操作，定义`queryWithPage`方法，用于分页查询。通过`ExceptionCodeManager`描述执行状态错误码等。JPA 应用的"Service"可继承`SimpleJpaService`。

#### 3.3 Dao 层

#### 3.4 Controller 层

默认提供三种"Controller(使用`Scanned`注解)"代理类:

* 注入用户数据到请求参数中, 即数据隔离，实现类为`ApiProcessingInterceptor`
* 请求日志打印，实现类为`LoggingInterceptor`
* 参数校验，实现类为`ValidateInterceptor`

##### 3.4.1 Json解析

`ResponseBodyMessageConverter`支持把`Result`, `Number`, `String`, `Result.ResultState`等对象转为`Message`对象，即可保证接口返回的数据统一为`Message`规定的格式(), 应用可调用`Message.of()`方法创建实例。


#### 3.5 异常处理

##### 3.5.1 默认处理异常类型

`GlobalHandlerExceptionResolver`处理在请求中发生的异常，包括如下异常:

* ServiceException
* NullPointerException || IndexOutOfBoundsException || NoSuchElementException
* HttpMessageNotReadableException
* UnsupportedOperationException
* MethodArgumentNotValidException
* DuplicateKeyException

##### 3.5.2 描述信息

应用中的描述信息统一由 `ExceptionCodeManager` 处理。`ExceptionCodeDescriptor` 封装了描述信息，默认提供了两种类型:

* SimpleExceptionCodeDescriptor
* ResourceExceptionCodeDescriptor

`SimpleExceptionCodeDescriptor`为默认配置，包含了常见的描述代码和描述信息，例如操作成功，操作失败等。

`ResourceExceptionCodeDescriptor` 从"messages"文件总读取配置的描述信息。
