---
layout: post
category : Tech
title : Mybatis 插入数据时返回自增主键
tags : [java, notes]
---
{% include JB/setup %}

开发中，有时需要获取自增主键的id。如果直接操作数据库，可通过`SELECT LAST_INSERT_ID();`来获取最近插入数据的自增id。使用ORM框架的话，就需要做其他工作了，幸好Mybatis已经内置了此功能。

Mybatis通过JDBC的`getGeneratedKeys`来获取insert时产生的主键。只需在mapper文件的`insert`节点上添加`useGeneratedKeys`和`keyProperty`即可。需要注意的是，keyProperty的值为传入参数的属性名，Mybatis会自动把自增id的值赋给该属性， 而返回值仍为影响行数。

官方文档：

> useGeneratedKeys:
> (insert and update only) This tells MyBatis to use the JDBC getGeneratedKeys method to retrieve keys generated internally by the database (e.g. auto increment fields in RDBMS like MySQL or SQL Server). Default: false
>
> keyProperty:
>
> (insert and update only) Identifies a property into which MyBatis will set the key value returned by getGeneratedKeys, or by a selectKey child element of the insert statement. Default: unset. Can be a comma separated list of property names if multiple generated columns are expected.

如下:

```xml
<insert id="insertAuthor" keyProperty="id" useGeneratedKeys="true">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>
```

也可通过全局配置启用`getGeneratedKeys`功能:

```xml
<bean class="org.apache.ibatis.session.Configuration" id="configuration">
	<property name="useGeneratedKeys" value="true" />
</bean>
<bean id="sqlSessonFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="configuration" ref="configuration" />
  	......
</bean>
```

除了以上方法外，还可通过`selectKey`实现:

```xml
<selectKey
  keyProperty="id"
  resultType="int"
  order="BEFORE"
  statementType="PREPARED">
  ......
</selectKey>
```

下面我们来看看该功能是如何实现的。

`PreparedStatementHandler`:

```java
@Override
public int update(Statement statement) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();
    int rows = ps.getUpdateCount();
    Object parameterObject = boundSql.getParameterObject();
    // 创建keyGenerator
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
    keyGenerator.processAfter(executor, mappedStatement, ps, parameterObject);
    return rows;
}
```

`MappedStatement`:

```java
public Builder(Configuration configuration, String id, SqlSource sqlSource, SqlCommandType sqlCommandType) {
      ......
      // 读取配置，来决定创建什么类型的keyGenerator
      mappedStatement.keyGenerator = configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType) ? new Jdbc3KeyGenerator() : new NoKeyGenerator();
      ......
    }
```

`Jdbc3KeyGenerator`:

```java
public void processBatch(MappedStatement ms, Statement stmt, Collection<Object> parameters) {
  ResultSet rs = null;
  try {
    // 读取生成的id
    rs = stmt.getGeneratedKeys();
    final Configuration configuration = ms.getConfiguration();
    final TypeHandlerRegistry typeHandlerRegistry = configuration.getTypeHandlerRegistry();
    final String[] keyProperties = ms.getKeyProperties();
    final ResultSetMetaData rsmd = rs.getMetaData();
    TypeHandler<?>[] typeHandlers = null;
    // 读取定义的keyProperty， 根据查询结果创建typeHandlers
    if (keyProperties != null && rsmd.getColumnCount() >= keyProperties.length) {
      for (Object parameter : parameters) {
        // there should be one row for each statement (also one for each parameter)
        if (!rs.next()) {
          break;
        }
        final MetaObject metaParam = configuration.newMetaObject(parameter);
        if (typeHandlers == null) {
          typeHandlers = getTypeHandlers(typeHandlerRegistry, metaParam, keyProperties, rsmd);
        }
        // 根据结果来赋值
        populateKeys(rs, metaParam, keyProperties, typeHandlers);
      }
    }
  } catch (Exception e) {
    throw new ExecutorException("Error getting generated key or setting result to parameter object. Cause: " + e, e);
  } finally {
    if (rs != null) {
      try {
        rs.close();
      } catch (Exception e) {
        // ignore
      }
    }
  }
}
```

```java
// 根据ResultSet和keyProperties来赋值
private void populateKeys(ResultSet rs, MetaObject metaParam, String[] keyProperties, TypeHandler<?>[] typeHandlers) throws SQLException {
    for (int i = 0; i < keyProperties.length; i++) {
        TypeHandler<?> th = typeHandlers[i];
        if (th != null) {
          Object value = th.getResult(rs, i + 1);
          metaParam.setValue(keyProperties[i], value);
        }
    }
}
```

`MetaObject`:

```java
// 利用反射来给具体属性赋值
public void setValue(String name, Object value) {
  PropertyTokenizer prop = new PropertyTokenizer(name);
  if (prop.hasNext()) {
    MetaObject metaValue = metaObjectForProperty(prop.getIndexedName());
    if (metaValue == SystemMetaObject.NULL_META_OBJECT) {
      if (value == null && prop.getChildren() != null) {
        // don't instantiate child path if value is null
        return;
      } else {
        metaValue = objectWrapper.instantiatePropertyValue(name, prop, objectFactory);
      }
    }
    metaValue.setValue(prop.getChildren(), value);
  } else {
    objectWrapper.set(prop, value);
  }
}
```
