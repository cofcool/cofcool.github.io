---
layout: post
category: Tech
title: Mybaits执行查询时抛出 ExecutorException No constructor found 问题解析
tags: [java, mybatis, sourcode]
excerpt: Mybatis执行查询时抛出 org.mybatis.spring.MyBatisSystemException nested exception is org.apache.ibatis.executor.ExecutorException No constructor found，到底是什么原因导致的？
---

{% include JB/setup %}

Mybatis使用`ResultSetHandler`负责处理执行结果，默认实现为`DefaultResultSetHandler`。

执行查询语句映射查询结果到Java Bean时，会根据resultMap配置和映射类的定义来解析为具体的对象，并根据以下几种条件进行解析。

* resultMap的子元素配置typeHandler
* resultMap包含constructor元素
* 映射类为接口或定义无参构造方法，也就是默认构造方法
* 映射类定义有参构造方法
* 开启自动映射

如不满足以上几种情况则会抛出`ExecutorException`异常，一般来说，我们在开发中通过调用无参构造方法的方式来创建结果实例，但是如果定义了有参构造方法，那情况就有所不同了。映射类定义有参构造方法之后，Mybatis会按照`映射类定义有参构造方法`的方式进行解析，这时它会把`result`映射的全部结果作为构造方法的参数来创建实例，也就是说映射字段和构造方法参数必须一致，如不一致则抛出`ExecutorException`异常，即未找到合适的构造方法。

```java
throw new ExecutorException("No constructor found in " + resultType.getName() + " matching " + rsw.getClassNames());
```

下面我们来看看Mybatis是如何处理这个过程的。

**DefaultResultSetHandler**，ResultSetHandler的默认实现，DefaultSqlSession调用它来解析查询结果，具体过程如下代码所示。

```java
// 处理查询结果
// 调用私有方法handleResultSet来解析每一个resultMap
public List<Object> handleResultSets(Statement stmt) throws SQLException {
    ErrorContext.instance().activity("handling results").object(mappedStatement.getId());

    final List<Object> multipleResults = new ArrayList<Object>();

    int resultSetCount = 0;
    ResultSetWrapper rsw = getFirstResultSet(stmt);

    List<ResultMap> resultMaps = mappedStatement.getResultMaps();
    int resultMapCount = resultMaps.size();
    validateResultMapsCount(rsw, resultMapCount);

    // 处理resultMap
    while (rsw != null && resultMapCount > resultSetCount) {
      ResultMap resultMap = resultMaps.get(resultSetCount);
      handleResultSet(rsw, resultMap, multipleResults, null);
      rsw = getNextResultSet(stmt);
      cleanUpAfterHandlingResultSet();
      resultSetCount++;
    }

    // 处理resultSets
    String[] resultSets = mappedStatement.getResultSets();
    if (resultSets != null) {
      while (rsw != null && resultSetCount < resultSets.length) {
        ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
        if (parentMapping != null) {
          String nestedResultMapId = parentMapping.getNestedResultMapId();
          ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
          handleResultSet(rsw, resultMap, null, parentMapping);
        }
        rsw = getNextResultSet(stmt);
        cleanUpAfterHandlingResultSet();
        resultSetCount++;
      }
    }

    return collapseSingleResultList(multipleResults);
}

// 创建查询结果实例
private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, List<Class<?>> constructorArgTypes, List<Object> constructorArgs, String columnPrefix)
    throws SQLException {
    final Class<?> resultType = resultMap.getType();
    final MetaClass metaType = MetaClass.forClass(resultType, reflectorFactory);
    final List<ResultMapping> constructorMappings = resultMap.getConstructorResultMappings();
    
    // resultMap的子元素配置 typeHandler
    if (hasTypeHandlerForResultObject(rsw, resultType)) {
        return createPrimitiveResultObject(rsw, resultMap, columnPrefix);
    } 
    // resultMap中包含constructor元素
    else if (!constructorMappings.isEmpty()) { 
        return createParameterizedResultObject(rsw, resultType, constructorMappings, constructorArgTypes, constructorArgs, columnPrefix);
    } 
    // 类型为接口或定义无参构造方法
    else if (resultType.isInterface() || metaType.hasDefaultConstructor()) { 
        return objectFactory.create(resultType);
    } 
    // 开启自动映射(默认开启)
    else if (shouldApplyAutomaticMappings(resultMap, false)) { 
        return createByConstructorSignature(rsw, resultType, constructorArgTypes, constructorArgs, columnPrefix);
    }
    throw new ExecutorException("Do not know how to create an instance of " + resultType);
}

// 根据constructor标签定义的元素进行值映射，然后通过ObjectFactory创建实例
Object createParameterizedResultObject(ResultSetWrapper rsw, Class<?> resultType, List<ResultMapping> constructorMappings,
      List<Class<?>> constructorArgTypes, List<Object> constructorArgs, String columnPrefix) {
    boolean foundValues = false;
    for (ResultMapping constructorMapping : constructorMappings) {
      final Class<?> parameterType = constructorMapping.getJavaType();
      final String column = constructorMapping.getColumn();
      final Object value;
      try {
        if (constructorMapping.getNestedQueryId() != null) {
          value = getNestedQueryConstructorValue(rsw.getResultSet(), constructorMapping, columnPrefix);
        } else if (constructorMapping.getNestedResultMapId() != null) {
          final ResultMap resultMap = configuration.getResultMap(constructorMapping.getNestedResultMapId());
          value = getRowValue(rsw, resultMap);
        } else {
          final TypeHandler<?> typeHandler = constructorMapping.getTypeHandler();
          value = typeHandler.getResult(rsw.getResultSet(), prependPrefix(column, columnPrefix));
        }
      } catch (ResultMapException e) {
        throw new ExecutorException("Could not process result for mapping: " + constructorMapping, e);
      } catch (SQLException e) {
        throw new ExecutorException("Could not process result for mapping: " + constructorMapping, e);
      }
      constructorArgTypes.add(parameterType);
      constructorArgs.add(value);
      foundValues = value != null || foundValues;
    }
    return foundValues ? objectFactory.create(resultType, constructorArgTypes, constructorArgs) : null;
}

// 根据构造方法创建实例，如未找到合适的构造方法，抛出异常
private Object createByConstructorSignature(ResultSetWrapper rsw, Class<?> resultType, List<Class<?>> constructorArgTypes, List<Object> constructorArgs, String columnPrefix) throws SQLException {
    for (Constructor<?> constructor : resultType.getDeclaredConstructors()) {
        if (typeNames(constructor.getParameterTypes()).equals(rsw.getClassNames())) {
            boolean foundValues = false;
            for (int i = 0; i < constructor.getParameterTypes().length; i++) {
                Class<?> parameterType = constructor.getParameterTypes()[i];
                String columnName = rsw.getColumnNames().get(i);
                TypeHandler<?> typeHandler = rsw.getTypeHandler(parameterType, columnName);
                Object value = typeHandler.getResult(rsw.getResultSet(), prependPrefix(columnName, columnPrefix));
                constructorArgTypes.add(parameterType);
                constructorArgs.add(value);
                foundValues = value != null || foundValues;
            }
            return foundValues ? objectFactory.create(resultType, constructorArgTypes, constructorArgs) : null;
        }
    }
    throw new ExecutorException("No constructor found in " + resultType.getName() + " matching " + rsw.getClassNames());
}
```

上文通过调用`MetaClass`的hasDefaultConstructor方法判断是否定义无参构造方法。

```java
// 是否有无参构造方法
public boolean hasDefaultConstructor() {
    return reflector.hasDefaultConstructor();
}
```

hasDefaultConstructor方法调用`Reflector`的hasDefaultConstructor方法，该方法判断defaultConstructor是否为NULL。

```java
// 为defaultConstructor赋值
// 也就是寻找无参构造方法
private void addDefaultConstructor(Class<?> clazz) {
    Constructor<?>[] consts = clazz.getDeclaredConstructors();
    for (Constructor<?> constructor : consts) {
        // 参数个数为0
        if (constructor.getParameterTypes().length == 0) {
            if (canAccessPrivateMethods()) {
                try {
                constructor.setAccessible(true);
                } catch (Exception e) {
                // Ignored. This is only a final precaution, nothing we can do.
                }
            }
            if (constructor.isAccessible()) {
                this.defaultConstructor = constructor;
            }
        }
    }
}

public boolean hasDefaultConstructor() {
    return defaultConstructor != null;
}
```