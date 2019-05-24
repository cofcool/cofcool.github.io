---
layout: post
category: Tech
title: Spring 项目自定义特定 Logger 的日志级别
tags: [java,spring]
excerpt: 如何关闭 PoolingHttpClientConnectionManager 的"DEBUG"日志
---

{% include JB/setup %}

在使用“httpclient”组件的项目中，当项目的日志级别为"DEBUG"时它会持续输出连接超时关闭的日志，干扰正常业务日志，因此需要关闭`PoolingHttpClientConnectionManager`的"DEBUG"日志。

以“log4j-1.2.17”为例，如下配置即可。

```properties
log4j.logger.org.apache.http.impl.conn.PoolingHttpClientConnectionManager=INFO
```

接下来我们来看看为什么如此配置。

log4j 通过`PropertyConfigurator`类载入配置文件。

```java
// 配置文件的 logger 前缀
static final String      LOGGER_PREFIX   = "log4j.logger.";

// 解析配置元素
protected void parseCatsAndRenderers(Properties props, LoggerRepository hierarchy) {
    Enumeration enumeration = props.propertyNames();
    while(enumeration.hasMoreElements()) {
        String key = (String) enumeration.nextElement();
        if(key.startsWith(CATEGORY_PREFIX) || key.startsWith(LOGGER_PREFIX)) {
        String loggerName = null;
        if(key.startsWith(CATEGORY_PREFIX)) {
            loggerName = key.substring(CATEGORY_PREFIX.length());
        } else if(key.startsWith(LOGGER_PREFIX)) {
            // 解析配置的 loggerName
            loggerName = key.substring(LOGGER_PREFIX.length());
        }
        String value =  OptionConverter.findAndSubst(key, props);
        Logger logger = hierarchy.getLogger(loggerName, loggerFactory);
        
        // ...
        }
    }
}
```

从上可得出可通过"loggerName"配置该"logger"的日志级别。`PoolingHttpClientConnectionManager`类通过`LogFactory.getLog(getClass())`获取"logger"实例，"loggerName"即为该类的类全限定名。所以，以上配置即可。

对于"Spring Boot"项目，可以如下配置:

```properties
# 定义日志组
logging.group.httpLog=org.apache.http.impl.conn.PoolingHttpClientConnectionManager
# 设置该组的输出级别
logging.level.httpLog=info
```

**代码解析**:

`LoggingApplicationListener`类负责解析配置。

```java
// 日志级别, 配置项前缀
private static final ConfigurationPropertyName LOGGING_LEVEL = ConfigurationPropertyName
			.of("logging.level");

// 日志组, 配置项前缀
private static final ConfigurationPropertyName LOGGING_GROUP = ConfigurationPropertyName
			.of("logging.group");

// 解析日志相关配置
protected void setLogLevels(LoggingSystem system, Environment environment) {
    if (!(environment instanceof ConfigurableEnvironment)) {
        return;
    }
    Binder binder = Binder.get(environment);
    Map<String, String[]> groups = getGroups();
    binder.bind(LOGGING_GROUP, STRING_STRINGS_MAP.withExistingValue(groups));
    Map<String, String> levels = binder.bind(LOGGING_LEVEL, STRING_STRING_MAP)
            .orElseGet(Collections::emptyMap);
    // 配置日志组的输出级别
    levels.forEach((name, level) -> {
        String[] groupedNames = groups.get(name);
        if (ObjectUtils.isEmpty(groupedNames)) {
            setLogLevel(system, name, level);
        }
        else {
            setLogLevel(system, groupedNames, level);
        }
    });
}
```