---
layout: post
category: Tech
title: 简述 Hibernate 拦截器和事件监听
tags: [java,sourcecode,spring]
excerpt: Hibernate 提供了 Interceptor 和 EventListener 让应用可以介入和了解它的执行过程。
---

{% include JB/setup %}

Hibernate 作为一个成熟的“ORM”框架，提供了`Interceptor`和`EventListener`让应用可以介入和了解它的执行过程。


<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [配置](#配置)
* [源码解析](#源码解析)

<!-- /code_chunk_output -->


## 配置

以"Spring Boot"项目(spring-boot-starter-data-jpa-2.1.0.RELEASE)为例:

`JpaProperties`的`properties`属性负责存储"JPA"的相关配置，如下所示:

```
spring.jpa.properties.hibernate.session_factory.interceptor=xx.MyInterceptor
spring.jpa.properties.hibernate.ejb.event.load=xx.MyLoadEventListener
```

MyLoadEventListener，实现`LoadEventListener`接口，也就是说可以监听"load"事件。
```java
public class MyLoadEventListener implements LoadEventListener {

    private static final long serialVersionUID = -7776598890958220332L;

    @Override
    public void onLoad(LoadEvent event, LoadType loadType) throws HibernateException {
        System.out.println(event.getEntityClassName());
    }
}
```
MyInterceptor，实现`Interceptor`接口，使用时直接继承`EmptyInterceptor`即可，可根据需求实现对应的方法，本例实现`onPrepareStatement`方法，即在预编译SQL时会调用该方法(建议使用`StatementInspector`取代)。
```java
public class MyInterceptor extends EmptyInterceptor {

    private static final long serialVersionUID = 304244397266301266L;

    @Override
    public String onPrepareStatement(String sql) {
        return super.onPrepareStatement(sql);
    }
}
```

## 源码解析

`SessionFactoryImpl`负责解析项目中的相关配置，session创建等，当然也包括 Interceptor 和 EventListener。

SessionFactoryImpl的内部类`SessionBuilderImpl`的`configuredInterceptor`方法创建"interceptor"，其中具体的解析操作由`SessionFactoryOptionsBuilder`(SessionFactoryOptions)完成。

```java
public static Interceptor configuredInterceptor(Interceptor interceptor, SessionFactoryOptions options) {
    // NOTE : DO NOT return EmptyInterceptor.INSTANCE from here as a "default for the Session"
    // 		we "filter" that one out here.  The return from here should represent the
    //		explicitly configured Interceptor (if one).  Return null from here instead; Session
    //		will handle it

    if ( interceptor != null && interceptor != EmptyInterceptor.INSTANCE ) {
        return interceptor;
    }

    // prefer the SF-scoped interceptor, prefer that to any Session-scoped interceptor prototype
    if ( options.getInterceptor() != null && options.getInterceptor() != EmptyInterceptor.INSTANCE ) {
        return options.getInterceptor();
    }

    // then check the Session-scoped interceptor prototype
    if ( options.getStatelessInterceptorImplementor() != null && options.getStatelessInterceptorImplementorSupplier() != null ) {
        throw new HibernateException(
                "A session scoped interceptor class or supplier are allowed, but not both!" );
    }
    else if ( options.getStatelessInterceptorImplementor() != null ) {
        try {
            /**
                * We could remove the getStatelessInterceptorImplementor method and use just the getStatelessInterceptorImplementorSupplier
                * since it can cover both cases when the user has given a Supplier<? extends Interceptor> or just the
                * Class<? extends Interceptor>, in which case, we simply instantiate the Interceptor when calling the Supplier.
                */
            return options.getStatelessInterceptorImplementor().newInstance();
        }
        catch (InstantiationException | IllegalAccessException e) {
            throw new HibernateException( "Could not supply session-scoped SessionFactory Interceptor", e );
        }
    }
    else if ( options.getStatelessInterceptorImplementorSupplier() != null ) {
        return options.getStatelessInterceptorImplementorSupplier().get();
    }

    return null;
}
```

`SessionFactoryImpl`的构造方法通过调用`prepareEventListeners`方法创建配置的"EventListener"。
```java
private void prepareEventListeners(MetadataImplementor metadata) {
    final EventListenerRegistry eventListenerRegistry = serviceRegistry.getService( EventListenerRegistry.class );
    final ConfigurationService cfgService = serviceRegistry.getService( ConfigurationService.class );
    final ClassLoaderService classLoaderService = serviceRegistry.getService( ClassLoaderService.class );

    eventListenerRegistry.prepare( metadata );

    for ( Map.Entry entry : ( (Map<?, ?>) cfgService.getSettings() ).entrySet() ) {
        if ( !String.class.isInstance( entry.getKey() ) ) {
            continue;
        }
        final String propertyName = (String) entry.getKey();
        if ( !propertyName.startsWith( org.hibernate.jpa.AvailableSettings.EVENT_LISTENER_PREFIX ) ) {
            continue;
        }
        final String eventTypeName = propertyName.substring(
                org.hibernate.jpa.AvailableSettings.EVENT_LISTENER_PREFIX.length() + 1
        );
        final EventType eventType = EventType.resolveEventTypeByName( eventTypeName );
        final EventListenerGroup eventListenerGroup = eventListenerRegistry.getEventListenerGroup( eventType );
        for ( String listenerImpl : ( (String) entry.getValue() ).split( " ," ) ) {
            eventListenerGroup.appendListener( instantiate( listenerImpl, classLoaderService ) );
        }
    }
}
```

其中，配置可参考以下类，它们定义了可配置项。

* org.hibernate.jpa.AvailableSettings
* org.hibernate.cfg.AvailableSettings
* org.hibernate.cfg.Environment