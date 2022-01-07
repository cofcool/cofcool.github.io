spring-data-mongo

## 环境安装

### 单机安装

[Install MongoDB Community Edition on Linux](https://docs.mongodb.com/manual/administration/install-on-linux/)

### 集群安装

MappingMongoConverter 转换对象
MongoTypeMapper


spring data mongo 自定义 convert

参考： `AbstractMongoConverter`: `instantiators`, `conversions`, `conversionService`; `MongoCustomConversions`

`org.springframework.data.mongodb.core.convert.AbstractMongoConverter#initializeConverters`, 注册 "conversions" 到 "conversionService" 中

```java
@Bean
public MappingMongoConverter mappingMongoConverter(MongoDatabaseFactory factory, MongoMappingContext context) {
    DbRefResolver dbRefResolver = new DefaultDbRefResolver(factory);
    MappingMongoConverter mappingConverter = new MappingMongoConverter(dbRefResolver, context);
     mappingConverter.setCustomConversions(new MongoCustomConversions(Collections.singletonList(EventDataConvert.INSTANCE)));
    Map<Class<?>, EntityInstantiator> customInstantiators = new HashMap<>();
    customInstantiators.put(EventData.class, new EntityInstantiator() {
        @Override
        public <T, E extends PersistentEntity<? extends T, P>, P extends PersistentProperty<P>> T createInstance(E entity, ParameterValueProvider<P> provider) {
            //noinspection unchecked
            return (T) new EventData("", "");
        }
    });
    mappingConverter.setInstantiators(new EntityInstantiators(customInstantiators));
    return mappingConverter;
}

enum EventDataConvert implements Converter<Document, EventData> {
    INSTANCE;
    @Override
    public EventData convert(Document source) {
        return new EventData(source.getString("source"), source.getString("type"), source.getLong("time"));
    }
}

```

mongoTemplate.findById(Query, Demo.class) 会陷入死循环直到栈溢出

org.springframework.data.mongodb.core.query.Criteria#criteriaChain 导致死循环

```java
public Criteria(String key) {
    this.criteriaChain = new ArrayList<Criteria>();
    this.criteriaChain.add(this);
    this.key = key;
}
```

criteriaChain持有自身因此导致遍历属性时死循环

MongoTemplate.stream() 方法，驱动内部的 `MongoIterable ` 定义遍历等操作，内部还是通过一次拉取多条数据然后让客户端遍历，具体数据量由服务器端决定

### 常见问题

#### 1. 更新数据时 updateFirst 阻塞

一台机器上有个三个分片，共有三台机器，数据根据主键进行 hash 分片，每个分片各有三个副本，当其中一台机宕机后，更新数据操作时线程一直处于等待服务器返回数据状态，猜测可能是主分片宕机后，节点选举出现脑裂问题，导致主分片并没有被选出来，更新操作一直处在等待主分片返回结果，可以增加仲裁节点

#### 2. 单个文档过大无法储存

压缩文档中较大的字段：

1. 自定义json序列化和反序列化/自定义数据-mongo binary 的 converter
2. 先把原对象转为json字符串再执行lz4压缩以及相应的反向操作

如不想添加新的数据结构，通过注解实现，可自定义 `org.springframework.data.mongodb.core.convert.MongoConverter` 和 `org.springframework.data.mapping.PersistentPropertyAccessor`, 另外实体属性描述参考 `org.springframework.data.mapping.model.AnnotationBasedPersistentProperty`