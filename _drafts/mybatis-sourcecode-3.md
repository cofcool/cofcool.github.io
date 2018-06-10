# Mybatis SQL执行源码分析 (四) 事务处理

基础环境：
* MyBatis 3.4.1
* mybatis-spring 1.3.0

Mybatis SQL执行源码分析系列文章：
* [Mybatis SQL执行源码分析 (一) 流程分析](./mybatis-sourcecode.md)
* [Mybatis SQL执行源码分析 (二) Mapper扫描及代理](./mybatis-sourcecode-1.md)
* [Mybatis SQL执行源码分析 (三) SqlSession创建](./mybatis-sourcecode-2.md)
* Mybatis SQL执行源码分析 (四) 事务处理
* [Mybatis SQL执行源码分析 (五) SQL执行](./mybatis-sourcecode-4.md)


MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和Java的POJOs映射成数据库中的记录。

本系列文章主要从源代码的角度解析`Mybatis`在`Spirng`框架上如何创建扫描，创建实例，以及SQL如何执行等核心功能。本文主要介绍Mybatis如何使用基于Spring配置的事务管理器和内置的事务管理器。

目录：

### 2.2 事务处理
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


