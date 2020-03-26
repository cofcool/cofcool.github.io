# shardingsphere 简单使用

```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>4.0.1</version>
</dependency>
```

```properties
spring.shardingsphere.datasource.name=master
spring.shardingsphere.datasource.master.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.master.drive-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.master.jdbc-url=jdbc:mysql://localhost:3306/my_test?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true
spring.shardingsphere.datasource.master.username=root
spring.shardingsphere.datasource.master.password=my_test
spring.shardingsphere.sharding.default-data-source-name=master

spring.shardingsphere.sharding.tables.vip_user.actual-data-nodes=master.vip_user${0..19}
spring.shardingsphere.sharding.tables.vip_user.table-strategy.inline.sharding-column=user_mobile
spring.shardingsphere.sharding.tables.vip_user.table-strategy.inline.algorithm-expression=vip_user${user_mobile % 20}

# 自动生成分布式 ID，插入数据时该字段不能作为插入项，JPA 可配置 @Column(insertable = false) 来忽略插入，让它自动生成
spring.shardingsphere.sharding.tables.vip_user.key-generator.type=SNOWFLAKE
spring.shardingsphere.sharding.tables.vip_user.key-generator.column=user_map_id
spring.shardingsphere.sharding.tables.vip_user.key-generator.props.worker.id=2
```

报错：
Failed to get driver instance for jdbcUrl=jdbc:mysql
java.sql.SQLException: No suitable driver

发现在`Tomcat`中启动时抛出该错误，但是通过`Spring Boot`的“Main”方法并不会报错。前者没有注册数据库驱动，但是后者注册了，没有找到二者的差别，`SPI`没有找到对应的数据库驱动，通过在系统变量中注册`jdbc.drivers`也未生效，容器还是找不到对应的数据驱动类，只能手动注册一次。

```java
static {
    try {
        Class.forName("com.mysql.jdbc.Driver");
    } catch (ClassNotFoundException e) {
        throw new IllegalStateException(e);
    }
}
```

执行`update`语句时，注意不要把分表ID作为修改字段，如 JPA 中配置 `@Column(updatable = false)`