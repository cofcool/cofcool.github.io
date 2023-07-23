---
layout: post
category : Tech
title : 简述 Jackson 默认类型机制
tags : [java]
excerpt: 开发通用接口时，参数序列化/反序列化是一个比较棘手的问题，参数类型不好确定。幸好 Jackson 提供 `DefaultTyping` 允许用户自定义序列化/反序列化对象类型。
---
{% include JB/setup %}

开发通用接口时，参数序列化/反序列化是一个比较棘手的问题，参数类型不好确定。简单直接办法是添加一个容器类型的对象，包裹真实参数，通过特殊接口获取真实类型后再进行转换。这样增加了开发复杂度和代码量，那有没有更简单的方法？

幸好 Jackson 提供 `DefaultTyping` 允许用户自定义序列化/反序列化对象类型，参考 `ObjectMapper#setDefaultTyping`，另外 `JsonTypeInfo` 注解也有类似功能。原理是序列化时读取对象类型（DefaultTyping），并把类型写入生成的 JSON 中，反序列化时读取类型字段获取类型并反序列化为该类型实例。*可能带来安全问题，谨慎使用。*

不过 `ObjectMapper#setDefaultTyping` 调用比较麻烦，推荐使用 `ObjectMapper#activateDefaultTypingAsProperty(PolymorphicTypeValidator, DefaultTyping, String)`。使用示例：

```java
// 提供 `getIoDataType` 属性
public interface DataType {

    String TYPE_KEY = "ioDataType";

    default String getIoDataType() {
        return getClass().getName();
    }

}

public record User(String username, String password) implements DataType {

}

// 激活 DefaultTyping
objectMapper.activateDefaultTypingAsProperty(
    objectMapper.getPolymorphicTypeValidator(), 
    ObjectMapper.DefaultTyping.NON_FINAL, 
    "ioDataType"
);

// 序列化
objectMapper.writeValueAsString(new User("xxxx", "123"))

//序列化后的 JSON 已包含 ioDataType 字段，对象类型为 com.example.demo.record.User
// {"username":"xxxx","password":"123","ioDataType":"com.example.demo.record.User"}
```


注意使用 Record 类型的数据时未把类型加入到结果中，原因是设置为 `DefaultTyping.NO_FINAL` ，而 Record 对象的字段都是 FINAL 修饰，所以无法生效。参考 `com.fasterxml.jackson.databind.ObjectMapper.DefaultTypeResolverBuilder#useForType`，改为使用 `EVERYTHING`，注意可能带来安全隐患。更安全的的方案是各个数据实体类都实现特定接口，提供 `getXxx` 以便提供类型属性。
