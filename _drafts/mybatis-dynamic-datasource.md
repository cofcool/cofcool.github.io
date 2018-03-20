# Spring Mybatis 实现动态数据源

SqlSessionFactoryBean
```java
public SqlSessionFactory getObject() throws Exception {
  if (this.sqlSessionFactory == null) {
    afterPropertiesSet();
  }

  return this.sqlSessionFactory;
}

@Override
public void afterPropertiesSet() throws Exception {
  notNull(dataSource, "Property 'dataSource' is required");
  notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");
  state((configuration == null && configLocation == null) || !(configuration != null && configLocation != null),
            "Property 'configuration' and 'configLocation' can not specified with together");

  this.sqlSessionFactory = buildSqlSessionFactory();
}

protected SqlSessionFactory buildSqlSessionFactory() throws IOException {

  Configuration configuration;

  XMLConfigBuilder xmlConfigBuilder = null;
  if (this.configuration != null) {
    configuration = this.configuration;
    if (configuration.getVariables() == null) {
      configuration.setVariables(this.configurationProperties);
    } else if (this.configurationProperties != null) {
      configuration.getVariables().putAll(this.configurationProperties);
    }
  } else if (this.configLocation != null) {
    xmlConfigBuilder = new XMLConfigBuilder(this.configLocation.getInputStream(), null, this.configurationProperties);
    configuration = xmlConfigBuilder.getConfiguration();
  } else {
    if (LOGGER.isDebugEnabled()) {
      LOGGER.debug("Property `configuration` or 'configLocation' not specified, using default MyBatis Configuration");
    }
    configuration = new Configuration();
    configuration.setVariables(this.configurationProperties);
  }
  ...

  // 配置基础环境，传入datasource
  configuration.setEnvironment(new Environment(this.environment, this.transactionFactory, this.dataSource));
  ...

  return this.sqlSessionFactoryBuilder.build(configuration);
}

```
SqlSessionFactory
```java
public interface SqlSessionFactory {

  SqlSession openSession();

  SqlSession openSession(boolean autoCommit);
  SqlSession openSession(Connection connection);
  SqlSession openSession(TransactionIsolationLevel level);

  SqlSession openSession(ExecutorType execType);
  SqlSession openSession(ExecutorType execType, boolean autoCommit);
  SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level);
  SqlSession openSession(ExecutorType execType, Connection connection);

  Configuration getConfiguration();

}
```

SqlSessionFactory的默认实现`DefaultSqlSessionFactory`，openSession方法调用`openSessionFromDataSource`函数来从数据源获取SqlSession实例。
```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
  Transaction tx = null;
  try {
    final Environment environment = configuration.getEnvironment();
    final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
    tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
    final Executor executor = configuration.newExecutor(tx, execType);
    return new DefaultSqlSession(configuration, executor, autoCommit);
  } catch (Exception e) {
    closeTransaction(tx); // may have fetched a connection so lets call close()
    throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```
SqlSession的默认实现为`DefaultSqlSession`，它的构造方法为`public DefaultSqlSession(Configuration configuration, Executor executor, boolean autoCommit)`，在创建实例的时候传入`Configuration`，调用如下代码获取`MappedStatement`实例。
```java
MappedStatement ms = configuration.getMappedStatement(statement);
```
