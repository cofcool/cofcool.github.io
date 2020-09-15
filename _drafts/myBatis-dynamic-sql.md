# MyBatis Dynamic SQL 

> SQL Generator for MyBatis or Spring JDBC Templates

详细信息可参考[官网](http://www.mybatis.org/mybatis-dynamic-sql/docs/introduction.html) 以及 [Github](https://github.com/mybatis/mybatis-dynamic-sql)。

```xml
<dependency>
  <groupId>org.mybatis.dynamic-sql</groupId>
  <artifactId>mybatis-dynamic-sql</artifactId>
  <version>1.2.0</version>
</dependency>
```

```plantuml
MapperAnnotationBuilder -> MapperAnnotationBuilder: parseStatement
MapperAnnotationBuilder -> MapperAnnotationBuilder: getAnnotationWrapper
MapperAnnotationBuilder -> MapperAnnotationBuilder: buildSqlSource
MapperAnnotationBuilder -> ProviderSqlSource: getBoundSql
ProviderSqlSource -> ProviderSqlSource: createSqlSource
ProviderSqlSource -> ProviderSqlSource: invokeProviderMethod
ProviderSqlSource -> MapperAnnotationBuilder
```

[generator](http://mybatis.org/generator/index.html)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <classPathEntry location="./h2-1.4.200.jar" />

    <context id="h2Test" targetRuntime="MyBatis3Kotlin">
        <jdbcConnection driverClass="org.h2.Driver"
                        connectionURL="jdbc:h2:tcp://localhost:9092/test"
                        userId=""
                        password="">
        </jdbcConnection>

        <javaModelGenerator targetPackage="com.example.demo.model" targetProject="src/main/kotlin">
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <javaClientGenerator type="XMLMAPPER" targetPackage="com.example.demo.dao"  targetProject="src/main/kotlin" />

        <table catalog="TEST" schema="TEST" tableName="USER" delimitIdentifiers="true"/>

    </context>
</generatorConfiguration>
```