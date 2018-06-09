# Mybatis SQL执行源码分析 (一) 流程分析

基础环境：
* MyBatis 3.4.1
* mybatis-spring 1.3.0

Mybatis SQL执行源码分析系列文章：
* Mybatis SQL执行源码分析 (一) 流程分析
* [Mybatis SQL执行源码分析 (二) SqlSession创建](./mybatis-sourcecode-1.md)
* [Mybatis SQL执行源码分析 (三) 事务处理](./mybatis-sourcecode-2.md)
* [Mybatis SQL执行源码分析 (四) SQL执行](./mybatis-sourcecode-3.md)

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和Java的POJOs映射成数据库中的记录。

本系列文章主要从源代码的角度解析`Mybatis`在`Spirng`框架上如何创建扫描，创建实例，以及SQL如何执行等核心功能。本文主要介绍Mybatis基本的执行流程。

目录：

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. Mapper扫描流程](#1-mapper扫描流程)
* [2. Myabtis代理DAO接口类过程](#2-myabtis代理dao接口类过程)
* [3. SqlSession创建流程](#3-sqlsession创建流程)
* [4. DAO执行流程](#4-dao执行流程)

<!-- /code_chunk_output -->

## 1. Mapper扫描流程
```plantuml
title: 扫描流程

MapperScannerConfigurer -> ClassPathMapperScanner: scan
ClassPathMapperScanner -> ClassPathMapperScanner: doScan
note left: 扫描配置的XML和Class，\n缓存对应关系，并把\n真实类替换为MapperFactoryBean，\n由它负责产生真正的类，\n也就是由MapperProxy代理的DAO接口类
```

由上文可知，MyBatis在扫描`Mapper`文件时。会把我们自己定义的接口类代理为MapperProxy，我们看看是如何执行代理过程的。

## 2. Myabtis代理DAO接口类过程

```plantuml
title: MapperProxy代理DAO接口类过程
MapperFactoryBean -> MapperFactoryBean: getObject()
MapperFactoryBean -> SqlSession: getSqlSession()
note left: ClassPathMapperScanner在创建\nMapperFactoryBean时，\n会把sqlSessionFactory,\nsqlSessionTemplate\n等参数传递给MapperFactoryBean，\n从而创建SqlSession(SqlSessionTemplate)，\n通过getObject返回代理类
SqlSession -> Configuration: getConfiguration()
Configuration -> MapperRegistry: getMapper()
MapperRegistry -> MapperProxyFactory: getMapper()
note left: 调用mapperRegistry\n的getMapper(type, sqlSession)\n，并由代理工厂类创建代理类实例
MapperProxyFactory -> MapperProxy: newInstance()
MapperProxy -> MapperFactoryBean
```
## 3. SqlSession创建流程

首先看看SqlSession的创建流程。
```plantuml
title: SqlSession创建流程
SqlSessionFactoryBean -> SqlSessionFactory: getObject()
note left: 实现FactoryBean接口，\n根据Configuration来\nbuildSqlSessionFactory，\n包括事务，DataSource等
SqlSessionFactory -> SqlSession: openSession()
note left: 默认实现为\nDefaultSqlSessionFactory，\n根据Configuration创建事务，\nExecutor等，并以此创建\nDefaultSqlSession实例
```

## 4. DAO执行流程

由“MapperProxy代理DAO接口类过程”可知，`MapperProxy`为真正的DAO接口实例，该类持有sqlSession，mapperInterface，sqlSession为`SqlSessionTemplate`实例，mapperInterface为定义的DAO接口类。
```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  // 如果是Object的方法，则直接调用
  if (Object.class.equals(method.getDeclaringClass())) {
    try {
      return method.invoke(this, args);
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
  }
  // 执行代理方法
  // MapperMethod封装了MethodSignature(方法参数，返回值等)和SqlCommand(Sql类型，方法ID)
  final MapperMethod mapperMethod = cachedMapperMethod(method);
  // 通过sqlSession执行sql
  return mapperMethod.execute(sqlSession, args);
}
```

如下图所示:
```plantuml
title: DAO执行流程

UserMapper -> MapperProxy: 查询用户: select
MapperProxy -> MapperMethod: invoke
note left: 代理扫描到的DAO接口
MapperMethod -> SqlSessionTemplate: execute
note left: 以执行query为例
SqlSessionTemplate -> this.sqlSessionProxy: select
note left: SqlSessionTemplate的\nsqlSessionProxy持有\nSqlSession接口的代理
this.sqlSessionProxy -> SqlSession: invoke
note left: 具体可查看SqlSessionTemplate的\n内部类SqlSessionInterceptor
SqlSession -> Executor: select
Executor -> Executor: query
note left: Executor负责具体的SQL执行，\n包含SIMPLE, REUSE, BATCH三种
Executor -> UserMapper
```

```plantuml
title: 创建SqlSession实例
SqlSessionFactoryBean --> SqlSessionFactory
SqlSessionFactoryBean --> DataSource
SqlSessionFactoryBean --> Configuration
SqlSessionFactoryBean --> TransactionFactory
SqlSessionFactoryBean --> SqlSessionFactoryBuilder

Configuration --> Environment
Configuration --> MapperRegistry
Configuration --> InterceptorChain

Environment --> DataSource
Environment --> TransactionFactory

TransactionFactory <|.. SpringManagedTransactionFactory
SpringManagedTransactionFactory ..> SpringManagedTransaction
SpringManagedTransaction --|> Transaction
SpringManagedTransaction --> Connection
SpringManagedTransaction --> DataSource

class SpringManagedTransaction {
  - openConnection()
}
note left: 调用Spring的DataSourceUtils\n.getConnection(this.dataSource)\n获取连接

Configuration <-- SqlSessionFactoryBuilder

SqlSessionFactory <.. SqlSessionFactoryBuilder
note bottom: 根据Configuration创建\nSqlSessionFactory
SqlSessionFactory ..> SqlSession
note bottom: 管理Sql执行和事务等

```
