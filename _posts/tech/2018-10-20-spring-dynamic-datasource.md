---
layout: post
category: Tech
title: 关于Spring动态数据源实现的简单讨论
tags: [java, spring]
excerpt: 在Java开发中，在业务中会同时访问多个数据库，也就是说需要多个 DataSource 实例，那如何实现多个数据源切换？

---

{% include JB/setup %}

在Java开发中，在业务中会同时访问多个数据库，也就是说需要多个 `DataSource` 实例。虽然直接创建实例也可，但从开发难易程度，代码实现是否优雅以及性能等角度来说，这么做弊大与利，幸运的是`Spring Jdbc`提供了API可简单实现动态切换数据源。

`AbstractRoutingDataSource`，实现了`javax.sql.DataSource`，继承它并实现`determineCurrentLookupKey`方法即可。定义多个数据源，以"key-value"的方式存储在`targetDataSources`中，`determineTargetDataSource`函数负责解析并获取当前数据源。源码如下:

```java
public abstract class AbstractRoutingDataSource extends AbstractDataSource implements InitializingBean {

    // 存储数据源
    @Nullable
    private Map<Object, Object> targetDataSources;

    // afterPropertiesSet 方法处理 targetDataSources 并转换为合适类型存储在 resolvedDataSources 中
    @Nullable
	private Map<Object, DataSource> resolvedDataSources;

    // 创建连接
    public Connection getConnection() throws SQLException {
    	return determineTargetDataSource().getConnection();
    }

    // 解析获取数据源
    protected DataSource determineTargetDataSource() {
        Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
        // 调用之类的实现获取 数据源 对应的key
        Object lookupKey = determineCurrentLookupKey();
        // 获取数据源
        DataSource dataSource = this.resolvedDataSources.get(lookupKey);
        if (dataSource == null && (this.lenientFallback || lookupKey == null)) {
        	dataSource = this.resolvedDefaultDataSource;
        }
        if (dataSource == null) {
        	throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");
        }
        return dataSource;
    }

    // 子类实现数据源切换逻辑
    protected abstract Object determineCurrentLookupKey();

    ...
}
```

自定义类继承`AbstractRoutingDataSource`，在`determineCurrentLookupKey`方法中进行数据源切换处理。

实现思路:

* 使用`ThreadLocal`存储当前请求线程的数据源信息，配合注解来实现数据源切换
* 通过线程通信的方式来获取数据源信息
* 设立数据源配置中心，项目自主拉取数据源

以上是一些实现思路，大家如有好的想法，望发邮件讨论。

配置示例:

```xml
 <bean id="dataSource" class="net.cofcool.test.spring.data.DynamicDataSource">
    <property name="targetDataSources">
        <map>
            <entry key="dataSource1" value-ref="dataSource1"/>
            <entry key="dataSource2" value-ref="dataSource2"/>
        </map>
    </property>
    <property name="defaultTargetDataSource" ref="dataSource1"/>
</bean>
```
