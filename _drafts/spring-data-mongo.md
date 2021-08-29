spring-data-mongo

MappingMongoConverter 转换对象
MongoTypeMapper


spring data mongo 自定义 convert

参考： `AbstractMongoConverter`: `instantiators`, `conversions`, `conversionService`; `MongoCustomConversions`

`org.springframework.data.mongodb.core.convert.AbstractMongoConverter#initializeConverters`, 注册 "conversions" 到 "conversionService" 中

```java
@Bean
public MappingMongoConverter mappingMongoConverter(MongoDatabaseFactory factory, MongoMappingContext context,
                                            MongoCustomConversions conversions) {
    DbRefResolver dbRefResolver = new DefaultDbRefResolver(factory);
    MappingMongoConverter mappingConverter = new MappingMongoConverter(dbRefResolver, context);
    mappingConverter.setCustomConversions(conversions);
    mappingConverter.setCustomConversions(new MongoCustomConversions(Collections.singletonList(SyncEventConvert.INSTANCE)));
    Map<Class<?>, EntityInstantiator> customInstantiators = new HashMap<>();
    customInstantiators.put(SyncEvent.class, new EntityInstantiator() {
        @Override
        public <T, E extends PersistentEntity<? extends T, P>, P extends PersistentProperty<P>> T createInstance(E entity, ParameterValueProvider<P> provider) {
            //noinspection unchecked
            return (T) new SyncEvent("", "");
        }
    });
    mappingConverter.setInstantiators(new EntityInstantiators(customInstantiators));
    return mappingConverter;
}

enum SyncEventConvert implements Converter<Document, SyncEvent> {
    INSTANCE;
    @Override
    public SyncEvent convert(Document source) {
        return new SyncEvent(source.getString("source"), source.getString("type"), source.getLong("time"));
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