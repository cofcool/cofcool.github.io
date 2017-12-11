---
layout: post
category : Tech
title : Jackson 自定义反序列化行为
tags : [java]
---
{% include JB/setup %}

Jackson具有很高的扩展性，如果默认实现满足不了需求，可以自由扩展。

我们来看看如何在反序列化时把空字符串解析为`NULL`。

示例代码如下：

```java
public ObjectMapper getObjectMapper() {
    ObjectMapper objectMapper = new ObjectMapper();
    // 忽略值为NULL的字段
    objectMapper.setSerializationInclusion(Include.NON_NULL);
    // 未知字段不会抛出异常
    objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

    // 添加反序列化类
    SimpleModule module = new SimpleModule();
    module.addDeserializer(String.class, new StringDeserializer());

    // 注册module
    objectMapper.registerModule(module);

    return objectMapper;
}

public class StringDeserializer extends StdDeserializer<String> {

    StringDeserializer() {
        this(null);
    }

    StringDeserializer(Class<?> clazz) {
        super(clazz);
    }

    @Override
    public String deserialize(JsonParser jp, DeserializationContext ctxt)
        throws IOException {
        JsonNode node = jp.getCodec().readTree(jp);
        if (node.getNodeType() == JsonNodeType.STRING) {
            String val = node.asText("");
            if (Strings.isNullOrEmpty(val.trim())) {
                return null;
            }
        }

        return node.asText();
    }
}
```

`SimpleModule` 官方描述如下:

> Vanilla {@link Module} implementation that allows registration
> of serializers and deserializers, bean serializer
> and deserializer modifiers, registration of subtypes and mix-ins
> as well as some other commonly
> needed aspects (addition of custom {@link AbstractTypeResolver}s,
> {@link com.fasterxml.jackson.databind.deser.ValueInstantiator}s).

通过该类我们hack它的序列化和反序列化的默认行为。

`ObjectMapper` 通过`registerModule`方法来注册创建的module，官方描述如下:

```java
/**
 * Method for registering a module that can extend functionality
 * provided by this mapper; for example, by adding providers for
 * custom serializers and deserializers.
 *
 * @param module Module to register
 */
public ObjectMapper registerModule(Module module)
```
