---
layout: post
category : Tech
title : ApplicationRunner VS CommandLineRunner
tags : [java, spring]
excerpt: ApplicationRunner 和 CommandLineRunner 本质上是同一类接口，除了参数有所差异外其余功能一致，那 Spring 为什么要设计为两种接口？
---
{% include JB/setup %}

`ApplicationRunner` 和 `CommandLineRunner` 本质上是同一类接口，除了参数有所差异外其余功能一致，那 Spring 为什么要设计为两种接口？

```java
public interface ApplicationRunner {

    void run(ApplicationArguments args) throws Exception;

}

public interface CommandLineRunner {

    void run(String... args) throws Exception;

}
```

`ApplicationRunner` 的 `run` 方法参数类型为 `ApplicationArguments`，由 Spring Boot 框架把传入的参数进行解析封装，使用更方便，如调用 `getOptionValues(String name)` 读取参数值。而 `CommandLineRunner` 的 `run` 方法参数类型为字符串数组，未做任何解析，还是原始参数，使用时还需要自己处理。

提供两种接口可以方便用户根据自己的需求进行扩展，更为灵活。

Spring Boot 项目程序启动最后一步会调用 Runner 实例，二者在同一阶段调用，功能定位一致，除了参数有所差异。
因为是同步调用，可能会阻止项目正常启动，因此不推荐使用它实现耗时操作，耗时操作可使用支持异步调用的 `@EventListener` 实现。

`SpringApplication` 在准备好上下文，Bean 创建，监听器启动等流程后执行 `callRunners` 来调用 Runner 实例。

```java
// SpringApplication
public ConfigurableApplicationContext run(String... args) {
    context = createApplicationContext();
    context.setApplicationStartup(this.applicationStartup);
    prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
    refreshContext(context);
    afterRefresh(context, applicationArguments);
    Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
    if (this.logStartupInfo) {
        new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), timeTakenToStartup);
    }
    listeners.started(context, timeTakenToStartup);
    // 运行 runner
    callRunners(context, applicationArguments);    
}
        
private void callRunners(ApplicationContext context, ApplicationArguments args) {
    List<Object> runners = new ArrayList<>();
    runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
    runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
    // 排序
    AnnotationAwareOrderComparator.sort(runners);
    for (Object runner : new LinkedHashSet<>(runners)) {
        // 遍历执行 ApplicationRunner
        if (runner instanceof ApplicationRunner applicationRunner) {
            callRunner(applicationRunner, args);
        }
        // 遍历执行 CommandLineRunner
        if (runner instanceof CommandLineRunner commandLineRunner) {
            callRunner(commandLineRunner, args);
        }
    }
}
```
