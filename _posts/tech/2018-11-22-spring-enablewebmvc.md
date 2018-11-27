---
layout: post
category: Tech
title: EnableWebMvc背后的原理
tags: [java, spring, sourcecode]
excerpt: EnableWebMvc注解在Spring Mvc项目中是如何作用的？
---

{% include JB/setup %}

`@EnableWebMvc`在Spring Mvc项目负责创建"Mvc"项目中相关组件以及载入应用配置等。

@EnableWebMvc源码如下:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

`@EnableWebMvc`通过`@Import`导入`DelegatingWebMvcConfiguration`类。DelegatingWebMvcConfiguration继承`WebMvcConfigurationSupport`，调用`setConfigurers`方法获取应用创建的`WebMvcConfigurer`实例，并通过以`WebMvcConfigurerComposite`来代理这些实例，从而获取应用自定义配置。

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

    // 通过 WebMvcConfigurerComposite 管理用户自定义的 WebMvcConfigurer
    private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

    @Autowired(required = false)
    public void setConfigurers(List<WebMvcConfigurer> configurers) {
       if (!CollectionUtils.isEmpty(configurers)) {
           this.configurers.addWebMvcConfigurers(configurers);
       }
    }

    @Override
    protected void configurePathMatch(PathMatchConfigurer configurer) {
    	this.configurers.configurePathMatch(configurer);
    }

    ...
}
```

DelegatingWebMvcConfiguration 的父类 WebMvcConfigurationSupport 负责创建"Mvc"需要的一些组件，如`ResourceUrlProvider`, `HandlerMapping`等。

```java
public class WebMvcConfigurationSupport implements ApplicationContextAware, ServletContextAware {

    ...

    @Bean
    public ResourceUrlProvider mvcResourceUrlProvider() {
    	ResourceUrlProvider urlProvider = new ResourceUrlProvider();
    	UrlPathHelper pathHelper = getPathMatchConfigurer().getUrlPathHelper();
    	if (pathHelper != null) {
    		urlProvider.setUrlPathHelper(pathHelper);
    	}
    	PathMatcher pathMatcher = getPathMatchConfigurer().getPathMatcher();
    	if (pathMatcher != null) {
    		urlProvider.setPathMatcher(pathMatcher);
    	}
    	return urlProvider;
    }

    @Bean
    @Nullable
    public HandlerMapping defaultServletHandlerMapping() {
    	Assert.state(this.servletContext != null, "No ServletContext set");
    	DefaultServletHandlerConfigurer configurer = new DefaultServletHandlerConfigurer(this.servletContext);
    	configureDefaultServletHandling(configurer);
    	return configurer.buildHandlerMapping();
    }

    // 由 ApplicationContextAwareProcessor 注入
    @Override
    public void setApplicationContext(@Nullable ApplicationContext applicationContext) {
    	this.applicationContext = applicationContext;
    }

    // 由 ServletContextAwareProcessor 注入
    @Override
    public void setServletContext(@Nullable ServletContext servletContext) {
        this.servletContext = servletContext;
    }

    ...

}
```

因此在web项目中只需使用`@EnableWebMvc`注解即可实现项目自动配置。

Spring Mvc 请求处理流程可参考[Spring MVC 自定义接口参数解析器](/tech/2017/12/19/spring-custom-param-resolve)。
