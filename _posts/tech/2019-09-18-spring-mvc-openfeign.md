---
layout: post
category : Tech
title : Spring MVC 集成 OpenFeign
tags : [java, spring]
excerpt: 使用 Feign 可以让网络请求像原生方法一样简单易用，让代码更美
---
{% include JB/setup %}

[Feign](https://github.com/OpenFeign/feign) 可以使网络请求变得及其简单，官方介绍:

> Feign is a Java to HTTP client binder inspired by Retrofit, JAXRS-2.0, and WebSocket. Feign's first goal was reducing the complexity of binding Denominator uniformly to HTTP APIs regardless of ReSTfulness.

使用它可以让网络请求像原生方法一样简单易用，更重要的是代码很美。你要是不信继续往下看:

```java
// 声明请求
interface GitHub {
  @RequestLine("GET /repos/{owner}/{repo}/contributors")
  List<Contributor> contributors(@Param("owner") String owner, @Param("repo") String repo);

  @RequestLine("POST /repos/{owner}/{repo}/issues")
  void createIssue(Issue issue, @Param("owner") String owner, @Param("repo") String repo);

}

public static class Contributor {
  String login;
  int contributions;
}

public static class Issue {
  String title;
  String body;
  List<String> assignees;
  int milestone;
  List<String> labels;
}

public class MyApp {
  public static void main(String... args) {
    // 创建 Github 实例
    GitHub github = Feign.builder()
                         .decoder(new GsonDecoder())
                         .target(GitHub.class, "https://api.github.com");
  
    // 发起网络请求
    List<Contributor> contributors = github.contributors("OpenFeign", "feign");
    for (Contributor contributor : contributors) {
      System.out.println(contributor.login + " (" + contributor.contributions + ")");
    }
  }
}
```

但是对于`Spring MVC`项目来说，这么简单好用的网络请求库却不能直接使用，因为官方只开发了`Spring Cloud`版本。不过作为开源项目，我们可以直接看到官方的代码，要修改适配的话那岂不是轻而易举。[Spring Cloud OpenFeign](https://spring.io/projects/spring-cloud-openfeign) 项目是官方为集成到 `Spring Boot` 而开发的，因此它依赖"Boot"相关组件，我们可以根据需求剔除这些组件，集成到"MVC"项目中, 代码可参考 [spring-mvc-openfeign](https://github.com/cofcool/chaos-server/tree/dev/spring-mvc-openfeign)。

思路:

1. 通过包扫描器扫描到使用 `@FeignClient` 注解的接口类
2. 注解的相关配置以及 `FeignClientProperties` 注入到 `Feign.Builder` 中
3. 通过 `Feign.Builder` 构建实例并放到 `Spring` 容器中
4. 通过`@Resource`等注解把实例注入到依赖的组件中

关键代码说明:

通过 `@EnableFeignClients`调用 `FeignClientsRegistrar`, `FeignClientsRegistrar`负责扫描及配置，由`FeignClientFactoryBean`创建对应的实例。

**FeignClientsRegistrar**

```java
public void registerFeignClients(AnnotationMetadata metadata,
    BeanDefinitionRegistry registry) {
    ClassPathScanningCandidateComponentProvider scanner = getScanner();
    scanner.setResourceLoader(this.resourceLoader);

    Set<String> basePackages;

    Map<String, Object> attrs = metadata
        .getAnnotationAttributes(EnableFeignClients.class.getName());
    // 寻找带有 FeignClient 注解的接口类
    AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(
        FeignClient.class);
    final Class<?>[] clients = attrs == null ? null
        : (Class<?>[]) attrs.get("clients");
    if (clients == null || clients.length == 0) {
        scanner.addIncludeFilter(annotationTypeFilter);
        basePackages = getBasePackages(metadata);
    }
    ...
}
```

**FeignClientFactoryBean**

```java
@Override
public Object getObject() throws Exception {
    FeignContext context = applicationContext.getBean(FeignContext.class);
    Feign.Builder builder = feign(context);

    if (StringUtils.hasText(this.url) && !this.url.startsWith("http")) {
        this.url = "http://" + this.url;
    }
    String url = this.url + cleanPath();
    Client client = getOptional(context, Client.class);
    if (client != null) {
        builder.client(client);
    }
    Targeter targeter = get(context, Targeter.class);
    // 创建实例
    return targeter.target(this, builder, context, new HardCodedTarget<>(
        this.type, this.name, url));
}
```

更多内容可参考官方文档和 [FeignTest](https://github.com/cofcool/chaos-server/blob/dev/spring-mvc-openfeign/src/test/java/test/FeignTest.java)。