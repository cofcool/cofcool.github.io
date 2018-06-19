---
layout: post
category: Tech
title: Mybatis SQL执行源码分析 (二) SQL执行
tags: [java, sourcode, mybatis]
excerpt: 本系列文章主要从源代码的角度解析Mybatis在Spirng框架上如何创建扫描，创建实例，以及SQL如何执行等核心功能。本文主要介绍SQL执行过程。
---

{% include JB/setup %}

基础环境：
* MyBatis 3.4.1
* mybatis-spring 1.3.0

Mybatis SQL执行源码分析系列文章：
* [Mybatis SQL执行源码分析 (一) Mapper扫描及代理](./mybatis-sourcecode-1.md)
* Mybatis SQL执行源码分析 (二) SQL执行


MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和Java的POJOs映射成数据库中的记录。

本系列文章主要从源代码的角度解析`Mybatis`在`Spirng`框架上如何创建扫描，创建实例，以及SQL如何执行等核心功能。本文主要介绍SQL执行过程。

目录：

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 流程](#1-流程)
	* [1.1 SqlSession创建流程](#11-sqlsession创建流程)
	* [1.2 DAO执行流程](#12-dao执行流程)
* [2. 代码分析](#2-代码分析)
	* [2.1 创建 SqlSession](#21-创建-sqlsession)
	* [2.2 SQL执行](#22-sql执行)
	* [2.3 事务处理](#23-事务处理)
		* [2.3.1 Mybatis管理](#231-mybatis管理)
		* [2.3.2 Spring管理](#232-spring管理)

<!-- /code_chunk_output -->

## 1. 流程

### 1.1 SqlSession创建流程

首先看看SqlSession的创建流程。
```plantuml
title: SqlSession创建流程
SqlSessionFactoryBean -> SqlSessionFactory: getObject()
note left: 实现FactoryBean接口，\n根据Configuration来\nbuildSqlSessionFactory，\n包括事务，DataSource等
SqlSessionFactory -> SqlSession: openSession()
note left: 默认实现为\nDefaultSqlSessionFactory，\n根据Configuration创建事务，\nExecutor等，并以此创建\nDefaultSqlSession实例
```

### 1.2 DAO执行流程

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

## 2. 代码分析

经过以上分析可了解到Myabtis的基本执行流程和相关类，下面我们来看具体的代码实现。

### 2.1 创建 SqlSession
SqlSessionFactoryBean
```java
// 获取 SqlSessionFactory
public SqlSessionFactory getObject() throws Exception {
  if (this.sqlSessionFactory == null) {
    afterPropertiesSet();
  }

  return this.sqlSessionFactory;
}

@Override
public void afterPropertiesSet() throws Exception {
  // 检查基础数据
  notNull(dataSource, "Property 'dataSource' is required");
  notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");
  state((configuration == null && configLocation == null) || !(configuration != null && configLocation != null),
            "Property 'configuration' and 'configLocation' can not specified with together");

  this.sqlSessionFactory = buildSqlSessionFactory();
}

// 创建 SqlSessionFactory
protected SqlSessionFactory buildSqlSessionFactory() throws IOException {

  Configuration configuration;

  XMLConfigBuilder xmlConfigBuilder = null;
  if (this.configuration != null) {
	// 配置 Properties
    configuration = this.configuration;
    if (configuration.getVariables() == null) {
      configuration.setVariables(this.configurationProperties);
    } else if (this.configurationProperties != null) {
      configuration.getVariables().putAll(this.configurationProperties);
    }
  } else if (this.configLocation != null) {
	// 读取Configuration的XML配置
    xmlConfigBuilder = new XMLConfigBuilder(this.configLocation.getInputStream(), null, this.configurationProperties);
    configuration = xmlConfigBuilder.getConfiguration();
  } else {
    if (LOGGER.isDebugEnabled()) {
      LOGGER.debug("Property `configuration` or 'configLocation' not specified, using default MyBatis Configuration");
    }
    configuration = new Configuration();
    configuration.setVariables(this.configurationProperties);
  }

  // 创建Bean的相关类
  if (this.objectFactory != null) {
    configuration.setObjectFactory(this.objectFactory);
  }

  if (this.objectWrapperFactory != null) {
    configuration.setObjectWrapperFactory(this.objectWrapperFactory);
  }
  ...

  // 配置事务工厂
  // 如果未手动配置，则使用Spring中配置的事务工厂
  if (this.transactionFactory == null) {
    this.transactionFactory = new SpringManagedTransactionFactory();
  }

  // 配置基础环境，传入datasource，事务工厂等
  configuration.setEnvironment(new Environment(this.environment, this.transactionFactory, this.dataSource));
  ...

  return this.sqlSessionFactoryBuilder.build(configuration);
}

@Override
public void onApplicationEvent(ApplicationEvent event) {
    if (failFast && event instanceof ContextRefreshedEvent) {
      // 检查statements，确保所有 mapper statement 解析完毕
      this.sqlSessionFactory.getConfiguration().getMappedStatementNames();
    }
}

```

SqlSessionFactory的默认实现`DefaultSqlSessionFactory`，openSession方法调用`openSessionFromDataSource`函数来从数据源获取SqlSession实例。
```java
// 创建 SqlSession
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
  Transaction tx = null;
  try {
    // 环境配置
    final Environment environment = configuration.getEnvironment();
    // 事务工厂
    final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
    // 创建事务
    tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
    // 创建 executor
    final Executor executor = configuration.newExecutor(tx, execType);
    // 返回 DefaultSqlSession
    return new DefaultSqlSession(configuration, executor, autoCommit);
  } catch (Exception e) {
    closeTransaction(tx);
    throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}

// 事务工厂
private TransactionFactory getTransactionFactoryFromEnvironment(Environment environment) {
  if (environment == null || environment.getTransactionFactory() == null) {
    // 默认为 ManagedTransactionFactory
    // ManagedTransaction 把事务管理交给容器处理
    // JdbcTransaction Mybatis直接管理事务
    return new ManagedTransactionFactory();
  }
  return environment.getTransactionFactory();
}
```

### 2.2 SQL执行

由“MapperProxy代理DAO接口类过程”可知，`MapperProxy`为真正的DAO接口实例，该类持有sqlSession，mapperInterface，sqlSession为`SqlSessionTemplate`实例，mapperInterface为定义的DAO接口类。

```java
public class MapperProxy<T> implements InvocationHandler, Serializable {

  private final SqlSession sqlSession;
  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethod> methodCache;

  ...

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

  private MapperMethod cachedMapperMethod(Method method) {
    MapperMethod mapperMethod = methodCache.get(method);
    if (mapperMethod == null) {
      mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
      methodCache.put(method, mapperMethod);
    }
    return mapperMethod;
  }
}
```

MapperMethod
```java
public class MapperMethod {

  // 封装方法名和类型
  private final SqlCommand command;
  // 封装方法参数，返回值等
  private final MethodSignature method;

  public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
    // Configuration封装了MappedStatement，以此创建SqlCommand，MethodSignature
    this.command = new SqlCommand(config, mapperInterface, method);
    this.method = new MethodSignature(config, mapperInterface, method);
  }

  // 执行
  // 根据SQL类型调用sqlSession对应的方法
  public Object execute(SqlSession sqlSession, Object[] args) {
      Object result;
      switch (command.getType()) {
        case INSERT: {
          // 插入数据
      	  Object param = method.convertArgsToSqlCommandParam(args);
          result = rowCountResult(sqlSession.insert(command.getName(), param));
          break;
        }
        ...
        // 查询
        case SELECT:
          if (method.returnsVoid() && method.hasResultHandler()) {
            executeWithResultHandler(sqlSession, args);
            result = null;
          } else if (method.returnsMany()) {
            result = executeForMany(sqlSession, args);
          } else if (method.returnsMap()) {
            result = executeForMap(sqlSession, args);
          } else if (method.returnsCursor()) {
            result = executeForCursor(sqlSession, args);
          } else {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = sqlSession.selectOne(command.getName(), param);
          }
          break;
        case FLUSH:
          result = sqlSession.flushStatements();
          break;
        default:
          throw new BindingException("Unknown execution method for: " + command.getName());
      }
      if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
        throw new BindingException("Mapper method '" + command.getName()
            + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
      }
      return result;
    }

    ...
}
```

SqlSessionTemplate管理真正SqlSession实例，当执行DAO方法时，代理类经过一系列调用会调用到SqlSessionTemplate的系列方法，SqlSessionTemplate的sqlSessionProxy代理了SqlSession的实例，在调用这些方法时实际会调用`SqlSessionTemplate`的内部类`SqlSessionInterceptor`的invoke方法。

```java
private class SqlSessionInterceptor implements InvocationHandler {
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    // 创建 SqlSession
    SqlSession sqlSession = getSqlSession(
        SqlSessionTemplate.this.sqlSessionFactory,
        SqlSessionTemplate.this.executorType,
        SqlSessionTemplate.this.exceptionTranslator);
    try {
      // 调用SqlSession的方法
      Object result = method.invoke(sqlSession, args);
      if (!isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
        // 提交事务
        sqlSession.commit(true);
      }
      return result;
    } catch (Throwable t) {
      Throwable unwrapped = unwrapThrowable(t);
      if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {
        // release the connection to avoid a deadlock if the translator is no loaded. See issue #22
        closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
        sqlSession = null;
        Throwable translated = SqlSessionTemplate.this.exceptionTranslator.translateExceptionIfPossible((PersistenceException) unwrapped);
        if (translated != null) {
          unwrapped = translated;
        }
      }
      throw unwrapped;
    } finally {
      if (sqlSession != null) {
        closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
      }
    }
  }
}
```

SqlSession的默认实现为`DefaultSqlSession`，它的构造方法为`public DefaultSqlSession(Configuration configuration, Executor executor, boolean autoCommit)`，在创建实例的时候传入`Configuration`。DefaultSqlSession在执行Sql时需要获取配置的动态sql以及相关参数等，`MappedStatement`封装了这些数据，可通过Configuration获取`MappedStatement`实例。

以DefaultSqlSession的selectList为例：
```java
@Override
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
  try {
    MappedStatement ms = configuration.getMappedStatement(statement);
    // executor
    return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

MappedStatement,该类包含了Mapper定义的方法执行时需要的各种参数定义和配置。
```java
public final class MappedStatement {
  // sql语句
  private String resource;
  private Configuration configuration;
  // 方法名
  private String id;
  private Integer fetchSize;
  private Integer timeout;
  // MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。
  private StatementType statementType;
  // 结果的类型。MyBatis 通常可以推算出来，但是为了更加确定写上也不会有什么问题。MyBatis 允许任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。
  private ResultSetType resultSetType;
  // Represents the content of a mapped statement read from an XML file or an annotation
  private SqlSource sqlSource;
  private Cache cache;
  private ParameterMap parameterMap;
  // resultMap 元素是 MyBatis 中最重要最强大的元素。它可以让你从 90% 的 JDBC ResultSets 数据提取代码中解放出来, 并在一些情形下允许你做一些 JDBC 不支持的事情。 实际上，在对复杂语句进行联合映射的时候，它很可能可以代替数千行的同等功能的代码。 ResultMap 的设计思想是，简单的语句不需要明确的结果映射，而复杂一点的语句只需要描述它们的关系就行了。
  private List<ResultMap> resultMaps;
  private boolean flushCacheRequired;
  private boolean useCache;
  private boolean resultOrdered;
  // sql类型，插入，查询等
  private SqlCommandType sqlCommandType;
  // 自增生成器
  private KeyGenerator keyGenerator;
  // 唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。
  private String[] keyProperties;
  // （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。
  private String[] keyColumns;
  private boolean hasNestedResultMaps;
  // 如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。
  private String databaseId;
  private Log statementLog;
  private LanguageDriver lang;
  private String[] resultSets;

  // 构建 MappedStatement
  public static class Builder {
    private MappedStatement mappedStatement = new MappedStatement();

    public Builder(Configuration configuration, String id, SqlSource sqlSource, SqlCommandType sqlCommandType) {
      mappedStatement.configuration = configuration;
      mappedStatement.id = id;
      mappedStatement.sqlSource = sqlSource;
      mappedStatement.statementType = StatementType.PREPARED;
      mappedStatement.parameterMap = new ParameterMap.Builder(configuration, "defaultParameterMap", null, new ArrayList<ParameterMapping>()).build();
      mappedStatement.resultMaps = new ArrayList<ResultMap>();
      mappedStatement.sqlCommandType = sqlCommandType;
      mappedStatement.keyGenerator = configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType) ? new Jdbc3KeyGenerator() : new NoKeyGenerator();
      String logId = id;
      if (configuration.getLogPrefix() != null) {
        logId = configuration.getLogPrefix() + id;
      }
      mappedStatement.statementLog = LogFactory.getLog(logId);
      mappedStatement.lang = configuration.getDefaultScriptingLanuageInstance();
    }

    ...

    public Builder resultMaps(List<ResultMap> resultMaps) {
      mappedStatement.resultMaps = resultMaps;
      for (ResultMap resultMap : resultMaps) {
        mappedStatement.hasNestedResultMaps = mappedStatement.hasNestedResultMaps || resultMap.hasNestedResultMaps();
      }
      return this;
    }

    ...

    public MappedStatement build() {
      assert mappedStatement.configuration != null;
      assert mappedStatement.id != null;
      assert mappedStatement.sqlSource != null;
      assert mappedStatement.lang != null;
      mappedStatement.resultMaps = Collections.unmodifiableList(mappedStatement.resultMaps);
      return mappedStatement;
    }
  }
  ...

  // BoundSql封装了xml或注解中定义的动态sql
  // sql，parameterMappings，parameterObject，additionalParameters，metaParameters;
  public BoundSql getBoundSql(Object parameterObject) {
    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings == null || parameterMappings.isEmpty()) {
      boundSql = new BoundSql(configuration, boundSql.getSql(), parameterMap.getParameterMappings(), parameterObject);
    }

    // check for nested result maps in parameter mappings (issue #30)
    for (ParameterMapping pm : boundSql.getParameterMappings()) {
      String rmId = pm.getResultMapId();
      if (rmId != null) {
        ResultMap rm = configuration.getResultMap(rmId);
        if (rm != null) {
          hasNestedResultMaps |= rm.hasNestedResultMaps();
        }
      }
    }

    return boundSql;
  }
}
```

获取到MappedStatement之后，`Executor`负责执行具体的sql。
```java
public interface Executor {

  ResultHandler NO_RESULT_HANDLER = null;

  int update(MappedStatement ms, Object parameter) throws SQLException;

  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey cacheKey, BoundSql boundSql) throws SQLException;

  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;

  <E> Cursor<E> queryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds) throws SQLException;

  List<BatchResult> flushStatements() throws SQLException;

  void commit(boolean required) throws SQLException;

  void rollback(boolean required) throws SQLException;

  CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql);

  boolean isCached(MappedStatement ms, CacheKey key);

  void clearLocalCache();

  void deferLoad(MappedStatement ms, MetaObject resultObject, String property, CacheKey key, Class<?> targetType);

  Transaction getTransaction();

  void close(boolean forceRollback);

  boolean isClosed();

  void setExecutorWrapper(Executor executor);

}
```
Configuration
```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
  executorType = executorType == null ? defaultExecutorType : executorType;
  executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
  Executor executor;
  if (ExecutorType.BATCH == executorType) {
    executor = new BatchExecutor(this, transaction);
  } else if (ExecutorType.REUSE == executorType) {
    executor = new ReuseExecutor(this, transaction);
  } else {
    executor = new SimpleExecutor(this, transaction);
  }
  if (cacheEnabled) {
    executor = new CachingExecutor(executor);
  }
  executor = (Executor) interceptorChain.pluginAll(executor);
  return executor;
}

// plugin
public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
    ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    return parameterHandler;
  }

  public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, ParameterHandler parameterHandler,
      ResultHandler resultHandler, BoundSql boundSql) {
    ResultSetHandler resultSetHandler = new DefaultResultSetHandler(executor, mappedStatement, parameterHandler, resultHandler, boundSql, rowBounds);
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    return resultSetHandler;
  }

  public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
  }
```

BaseExecutor
```java
public abstract class BaseExecutor implements Executor {
  protected Connection getConnection(Log statementLog) throws SQLException {
    // 获取 connection
    Connection connection = transaction.getConnection();
    if (statementLog.isDebugEnabled()) {
      return ConnectionLogger.newInstance(connection, statementLog, queryStack);
    } else {
      return connection;
    }
  }
}
```

类图如下所示:
SqlSession
```plantuml
interface SqlSession

SqlSession <|.. DefaultSqlSession
SqlSession <|.. SqlSessionTemplate
SqlSessionTemplate --> sqlSessionProxy: SqlSession
SqlSessionTemplate --> SqlSessionFactory
```
Executor
```plantuml
interface Executor

Executor <|.. BaseExecutor
BaseExecutor <|-- BatchExecutor
BaseExecutor <|-- SimpleExecutor
BaseExecutor <|-- ReuseExecutor
```

### 2.3 事务处理

#### 2.3.1 Mybatis管理
DefaultSqlSession
```java
@Override
public void commit() {
  commit(false);
}

@Override
public void commit(boolean force) {
  try {
    executor.commit(isCommitOrRollbackRequired(force));
    dirty = false;
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error committing transaction.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}

@Override
public void rollback() {
  rollback(false);
}

@Override
public void rollback(boolean force) {
  try {
    executor.rollback(isCommitOrRollbackRequired(force));
    dirty = false;
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error rolling back transaction.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}

@Override
public void close() {
  try {
    executor.close(isCommitOrRollbackRequired(false));
    closeCursors();
    dirty = false;
  } finally {
    ErrorContext.instance().reset();
  }
}
```
BaseExecutor
```java
@Override
public void close(boolean forceRollback) {
  try {
    try {
      rollback(forceRollback);
    } finally {
      if (transaction != null) {
        transaction.close();
      }
    }
  } catch (SQLException e) {
    // Ignore.  There's nothing that can be done at this point.
    log.warn("Unexpected exception on closing transaction.  Cause: " + e);
  } finally {
    transaction = null;
    deferredLoads = null;
    localCache = null;
    localOutputParameterCache = null;
    closed = true;
  }
}

@Override
public void commit(boolean required) throws SQLException {
  if (closed) {
    throw new ExecutorException("Cannot commit, transaction is already closed");
  }
  clearLocalCache();
  flushStatements();
  if (required) {
    transaction.commit();
  }
}

@Override
public void rollback(boolean required) throws SQLException {
  if (!closed) {
    try {
      clearLocalCache();
      flushStatements(true);
    } finally {
      if (required) {
        transaction.rollback();
      }
    }
  }
}
```

#### 2.3.2 Spring管理

MyBatis使用SpringManagedTransactionFactory来获取通过Spring配置的Transaction。
```java
@Override
public Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit) {
  // 创建 SpringManagedTransaction
  return new SpringManagedTransaction(dataSource);
}
```

SpringManagedTransaction
```java
private void openConnection() throws SQLException {
  // 通过DataSourceUtils获取connection
  this.connection = DataSourceUtils.getConnection(this.dataSource);
  this.autoCommit = this.connection.getAutoCommit();
  this.isConnectionTransactional = DataSourceUtils.isConnectionTransactional(this.connection, this.dataSource);

  if (LOGGER.isDebugEnabled()) {
    LOGGER.debug(
        "JDBC Connection ["
            + this.connection
            + "] will"
            + (this.isConnectionTransactional ? " " : " not ")
            + "be managed by Spring");
  }
}

@Override
public void commit() throws SQLException {
  if (this.connection != null && !this.isConnectionTransactional && !this.autoCommit) {
    if (LOGGER.isDebugEnabled()) {
      LOGGER.debug("Committing JDBC Connection [" + this.connection + "]");
    }
    this.connection.commit();
  }
}

@Override
public void rollback() throws SQLException {
  if (this.connection != null && !this.isConnectionTransactional && !this.autoCommit) {
    if (LOGGER.isDebugEnabled()) {
      LOGGER.debug("Rolling back JDBC Connection [" + this.connection + "]");
    }
    this.connection.rollback();
  }
}

@Override
public void close() throws SQLException {
  DataSourceUtils.releaseConnection(this.connection, this.dataSource);
}
```

DataSourceUtils，该类为spring-jdbc包中提供的一个工具类。DataSourceUtils提供了静态方法，可根据数据源获取JDBC Connections，包含spring管理的事务连接，也可在自己程序中使用该类。

> Helper class that provides static methods for obtaining JDBC Connections from a DataSource. Includes special support for Spring-managed transactional Connections, e.g. managed by DataSourceTransactionManager or org.springframework.transaction.jta.JtaTransactionManager.
Used internally by Spring's org.springframework.jdbc.core.JdbcTemplate, Spring's JDBC operation objects and the JDBC DataSourceTransactionManager. Can also be used directly in application code
```java
/**
 * Obtain a Connection from the given DataSource. Translates SQLExceptions into
 * the Spring hierarchy of unchecked generic data access exceptions, simplifying
 * calling code and making any exception that is thrown more meaningful.
 * <p>Is aware of a corresponding Connection bound to the current thread, for example
 * when using {@link DataSourceTransactionManager}. Will bind a Connection to the
 * thread if transaction synchronization is active, e.g. when running within a
 * {@link org.springframework.transaction.jta.JtaTransactionManager JTA} transaction).
 * @param dataSource the DataSource to obtain Connections from
 * @return a JDBC Connection from the given DataSource
 * @throws org.springframework.jdbc.CannotGetJdbcConnectionException
 * if the attempt to get a Connection failed
 * @see #releaseConnection
 */
public static Connection getConnection(DataSource dataSource) throws CannotGetJdbcConnectionException {
  try {
    return doGetConnection(dataSource);
  }
  catch (SQLException ex) {
    throw new CannotGetJdbcConnectionException("Could not get JDBC Connection", ex);
  }
}

/**
 * Actually obtain a JDBC Connection from the given DataSource.
 * Same as {@link #getConnection}, but throwing the original SQLException.
 * <p>Is aware of a corresponding Connection bound to the current thread, for example
 * when using {@link DataSourceTransactionManager}. Will bind a Connection to the thread
 * if transaction synchronization is active (e.g. if in a JTA transaction).
 * <p>Directly accessed by {@link TransactionAwareDataSourceProxy}.
 * @param dataSource the DataSource to obtain Connections from
 * @return a JDBC Connection from the given DataSource
 * @throws SQLException if thrown by JDBC methods
 * @see #doReleaseConnection
 */
public static Connection doGetConnection(DataSource dataSource) throws SQLException {
  Assert.notNull(dataSource, "No DataSource specified");

  ConnectionHolder conHolder = (ConnectionHolder) TransactionSynchronizationManager.getResource(dataSource);
  if (conHolder != null && (conHolder.hasConnection() || conHolder.isSynchronizedWithTransaction())) {
    conHolder.requested();
    if (!conHolder.hasConnection()) {
      logger.debug("Fetching resumed JDBC Connection from DataSource");
      conHolder.setConnection(dataSource.getConnection());
    }
    return conHolder.getConnection();
  }
  // Else we either got no holder or an empty thread-bound holder here.

  logger.debug("Fetching JDBC Connection from DataSource");
  Connection con = dataSource.getConnection();

  if (TransactionSynchronizationManager.isSynchronizationActive()) {
    logger.debug("Registering transaction synchronization for JDBC Connection");
    // Use same Connection for further JDBC actions within the transaction.
    // Thread-bound object will get removed by synchronization at transaction completion.
    ConnectionHolder holderToUse = conHolder;
    if (holderToUse == null) {
      holderToUse = new ConnectionHolder(con);
    }
    else {
      holderToUse.setConnection(con);
    }
    holderToUse.requested();
    TransactionSynchronizationManager.registerSynchronization(
        new ConnectionSynchronization(holderToUse, dataSource));
    holderToUse.setSynchronizedWithTransaction(true);
    if (holderToUse != conHolder) {
      TransactionSynchronizationManager.bindResource(dataSource, holderToUse);
    }
  }

  return con;
}
```


SqlSessionUtils
```java
public static void closeSqlSession(SqlSession session, SqlSessionFactory sessionFactory) {
  notNull(session, NO_SQL_SESSION_SPECIFIED);
  notNull(sessionFactory, NO_SQL_SESSION_FACTORY_SPECIFIED);

  SqlSessionHolder holder = (SqlSessionHolder) TransactionSynchronizationManager.getResource(sessionFactory);
  if ((holder != null) && (holder.getSqlSession() == session)) {
    if (LOGGER.isDebugEnabled()) {
      LOGGER.debug("Releasing transactional SqlSession [" + session + "]");
    }
    holder.released();
  } else {
    if (LOGGER.isDebugEnabled()) {
      LOGGER.debug("Closing non transactional SqlSession [" + session + "]");
    }
    session.close();
  }
}
```
