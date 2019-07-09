# Tomcat笔记


<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

- [ 1. Tomcat结构解析](#1-tomcat结构解析)
  - [ 1.1 Tomcat服务器核心组件](#11-tomcat服务器核心组件)
- [ 2. Session](#2-session)
  - [ 2.1 Session实现类](#21-session实现类)
- [ 3. Http](#3-http)
  - [ 3.1 HttpServlet](#31-httpservlet)
  - [ 3.1 Request](#31-request)
  - [ 3.2 Response](#32-response)
- [ Core](#core)
  - [ Executor](#executor)
  - [ Connector](#connector)
  - [ Host](#host)
    - [ JNDI](#jndi)
      - [ Resource](#resource)
- [ 资料引用](#资料引用)

<!-- /code_chunk_output -->


## 1. Tomcat结构解析

### 1.1 Tomcat服务器核心组件

组件层级如下所示：

* Server
  * Service
    * Connector
    * Engine
      * Host
        * Context

## 2. Session

### 2.1 Session实现类

1. StandardManager
2. PersistentManager
   1. FileStore
   2. JDBCStore

## 3. Http

### 3.1 HttpServlet

实现类为`org.apache.catalina.servlets.DefaultServlet`

### 3.1 Request

实现类为`org.apache.catalina.connector.Request`

### 3.2 Response

实现类为`org.apache.catalina.connector.Response`，底层 Servlet Request 实现。

## Core

### Executor
9.0
StandardThreadExecutor
```java
protected int maxThreads = 200;
```
### Connector

Connector
```java
package org.apache.catalina.connector;

/**
 * Implementation of a Coyote connector.
 */
public class Connector extends LifecycleMBeanBase  {

  /**
   * Defaults to using HTTP/1.1 NIO implementation.
   */
  public Connector() {
      this("org.apache.coyote.http11.Http11NioProtocol");
  }

  public Connector(String protocol) {
      boolean aprConnector = AprLifecycleListener.isAprAvailable() &&
              AprLifecycleListener.getUseAprConnector();

      if ("HTTP/1.1".equals(protocol) || protocol == null) {
          if (aprConnector) {
              protocolHandlerClassName = "org.apache.coyote.http11.Http11AprProtocol";
          } else {
              protocolHandlerClassName = "org.apache.coyote.http11.Http11NioProtocol";
          }
      } else if ("AJP/1.3".equals(protocol)) {
          if (aprConnector) {
              protocolHandlerClassName = "org.apache.coyote.ajp.AjpAprProtocol";
          } else {
              protocolHandlerClassName = "org.apache.coyote.ajp.AjpNioProtocol";
          }
      } else {
          protocolHandlerClassName = protocol;
      }

      // Instantiate protocol handler
      ProtocolHandler p = null;
      try {
          Class<?> clazz = Class.forName(protocolHandlerClassName);
          p = (ProtocolHandler) clazz.getConstructor().newInstance();
      } catch (Exception e) {
          log.error(sm.getString(
                  "coyoteConnector.protocolHandlerInstantiationFailed"), e);
      } finally {
          this.protocolHandler = p;
      }

      // Default for Connector depends on this system property
      setThrowOnFailure(Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE"));
  }
}
```

### Host

读取配置(`HostConfig`):

配置`Context`:

```java
if (context.getDocBase() != null) {
    File docBase = new File(context.getDocBase());
    if (!docBase.isAbsolute()) {
        docBase = new File(host.getAppBaseFile(), context.getDocBase());
    }
    ...
}
```

未配置`Context`:

```java
// default to appBase dir + name
// 未配置 Context 时从 ContextName 读取，即 “ROOT”
expandedDocBase = new File(host.getAppBaseFile(), cn.getBaseName());
```

#### JNDI

##### Resource

Note that the resource name (here, jdbc/EmployeeDB) must match the value specified in the web application deployment descriptor.

This example assumes that you are using the HypersonicSQL database JDBC driver. Customize the driverClassName and driverName parameters to match your actual database's JDBC driver and connection URL.

The configuration properties for Tomcat's standard data source resource factory (org.apache.tomcat.dbcp.dbcp2.BasicDataSourceFactory) are as follows:

    - driverClassName - Fully qualified Java class name of the JDBC driver to be used.
    - username - Database username to be passed to our JDBC driver.
    - password - Database password to be passed to our JDBC driver.
    - url - Connection URL to be passed to our JDBC driver. (For backwards compatibility, the property driverName is also recognized.)
    - initialSize - The initial number of connections that will be created in the pool during pool initialization. Default: 0
    - maxTotal - The maximum number of connections that can be allocated from this pool at the same time. Default: 8
    - minIdle - The minimum number of connections that will sit idle in this pool at the same time. Default: 0
    - maxIdle - The maximum number of connections that can sit idle in this pool at the same time. Default: 8
    - maxWaitMillis - The maximum number of milliseconds that the pool will wait (when there are no available connections) for a connection to be returned before throwing an exception. Default: -1 (infinite)

Some additional properties handle connection validation:

    - validationQuery - SQL query that can be used by the pool to validate connections before they are returned to the application. If specified, this query MUST be an SQL SELECT statement that returns at least one row.
    - validationQueryTimeout - Timeout in seconds for the validation query to return. Default: -1 (infinite)
    - testOnBorrow - true or false: whether a connection should be validated using the validation query each time it is borrowed from the pool. Default: true
    - testOnReturn - true or false: whether a connection should be validated using the validation query each time it is returned to the pool. Default: false

The optional evictor thread is responsible for shrinking the pool by removing any connections which are idle for a long time. The evictor does not respect minIdle. Note that you do not need to activate the evictor thread if you only want the pool to shrink according to the configured maxIdle property.

The evictor is disabled by default and can be configured using the following properties:

    - timeBetweenEvictionRunsMillis - The number of milliseconds between consecutive runs of the evictor. Default: -1 (disabled)
    - numTestsPerEvictionRun - The number of connections that will be checked for idleness by the evictor during each run of the evictor. Default: 3
    - minEvictableIdleTimeMillis - The idle time in milliseconds after which a connection can be removed from the pool by the evictor. Default: 30*60*1000 (30 minutes)
    - testWhileIdle - true or false: whether a connection should be validated by the evictor thread using the validation query while sitting idle in the pool. Default: false

Another optional feature is the removal of abandoned connections. A connection is called abandoned if the application does not return it to the pool for a long time. The pool can close such connections automatically and remove them from the pool. This is a workaround for applications leaking connections.

The abandoning feature is disabled by default and can be configured using the following properties:

    - removeAbandoned - true or false: whether to remove abandoned connections from the pool. Default: false
    - removeAbandonedTimeout - The number of seconds after which a borrowed connection is assumed to be abandoned. Default: 300
    - logAbandoned - true or false: whether to log stack traces for application code which abandoned a statement or connection. This adds serious overhead. Default: false

Finally there are various properties that allow further fine tuning of the pool behaviour:

    - defaultAutoCommit - true or false: default auto-commit state of the connections created by this pool. Default: true
    - defaultReadOnly - true or false: default read-only state of the connections created by this pool. Default: false
    - defaultTransactionIsolation - This sets the default transaction isolation level. Can be one of NONE, READ_COMMITTED, READ_UNCOMMITTED, REPEATABLE_READ, SERIALIZABLE. Default: no default set
    - poolPreparedStatements - true or false: whether to pool PreparedStatements and CallableStatements. Default: false
    - maxOpenPreparedStatements - The maximum number of open statements that can be allocated from the statement pool at the same time. Default: -1 (unlimited)
    - defaultCatalog - The name of the default catalog. Default: not set
    - connectionInitSqls - A list of SQL statements run once after a Connection is created. Separate multiple statements by semicolons (;). Default: no statement
    - connectionProperties - A list of driver specific properties passed to the driver for creating connections. Each property is given as name=value, multiple properties are separated by semicolons (;). Default: no properties
    - accessToUnderlyingConnectionAllowed - true or false: whether accessing the underlying connections is allowed. Default: false

For more details, please refer to the commons-dbcp documentation.

## 资料引用

1. 程序员突击:Tomcat原理与JavaWeb系统开发, 陈菁菁
