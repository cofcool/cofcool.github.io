# Mybatis SQL执行源码分析 (一) Mapper扫描及代理

基础环境：
* MyBatis 3.4.1
* mybatis-spring 1.3.0

Mybatis SQL执行源码分析系列文章：
* [Mybatis SQL执行源码分析 (一) 流程分析](./mybatis-sourcecode.md)
* Mybatis SQL执行源码分析 (二) Mapper扫描及代理
* [Mybatis SQL执行源码分析 (三) SqlSession创建](./mybatis-sourcecode-2.md)
* [Mybatis SQL执行源码分析 (四) 事务处理](./mybatis-sourcecode-3.md)
* [Mybatis SQL执行源码分析 (五) SQL执行](./mybatis-sourcecode-4.md)


MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和Java的POJOs映射成数据库中的记录。

本系列文章主要从源代码的角度解析`Mybatis`在`Spirng`框架上如何创建扫描，创建实例，以及SQL如何执行等核心功能。本文主要介绍Mybatis如何扫描Mapper建立接口和XML的映射关系。

目录：

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 配置](#1-配置)
* [2. 扫描](#2-扫描)
* [3. 代理](#3-代理)

<!-- /code_chunk_output -->


## 1. 配置

MapperScannerConfigurer
```java
public class MapperScannerConfigurer implements BeanDefinitionRegistryPostProcessor, InitializingBean, ApplicationContextAware, BeanNameAware {

  // 扫描包路径
  private String basePackage;

  private boolean addToConfig = true;

  // SqlSession相关对象
  private SqlSessionFactory sqlSessionFactory;
  private SqlSessionTemplate sqlSessionTemplate;
  private String sqlSessionFactoryBeanName;
  private String sqlSessionTemplateBeanName;

  private Class<? extends Annotation> annotationClass;

  // 扫描继承该接口的接口
  private Class<?> markerInterface;

  @Override
  public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    if (this.processPropertyPlaceHolders) {
      processPropertyPlaceHolders();
    }

    // 创建扫描器
    ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
    scanner.setAddToConfig(this.addToConfig);
    scanner.setAnnotationClass(this.annotationClass);
    scanner.setMarkerInterface(this.markerInterface);
    scanner.setSqlSessionFactory(this.sqlSessionFactory);
    scanner.setSqlSessionTemplate(this.sqlSessionTemplate);
    scanner.setSqlSessionFactoryBeanName(this.sqlSessionFactoryBeanName);
    scanner.setSqlSessionTemplateBeanName(this.sqlSessionTemplateBeanName);
    scanner.setResourceLoader(this.applicationContext);
    scanner.setBeanNameGenerator(this.nameGenerator);
    // 注册Filter
    scanner.registerFilters();
    // 扫描
    scanner.scan(StringUtils.tokenizeToStringArray(this.basePackage, ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS));
  }
}
```

## 2. 扫描

ClassPathMapperScanner
```java
public class ClassPathMapperScanner extends ClassPathBeanDefinitionScanner {

  // 是否添加mapper到Configuration
  private boolean addToConfig = true;

  private SqlSessionFactory sqlSessionFactory;
  private SqlSessionTemplate sqlSessionTemplate;
  private String sqlSessionTemplateBeanName;
  private String sqlSessionFactoryBeanName;

  // 扫描注解
  private Class<? extends Annotation> annotationClass;

  // DAO接口的父类
  private Class<?> markerInterface;

  // 创建被MapperProxy代理的DAO接口实例
  private MapperFactoryBean<?> mapperFactoryBean = new MapperFactoryBean<Object>();

  ...

  /**
   * 保证父类扫描器扫到正确的接口，包括所有接口或继承markerInterface的接口和使用annotationClass注解的接口
   */
  public void registerFilters() {
    // 默认扫描所有接口
    boolean acceptAllInterfaces = true;

    // 如果定义了annotationClass，则扫描使用该注解的接口
    if (this.annotationClass != null) {
      addIncludeFilter(new AnnotationTypeFilter(this.annotationClass));
      // 不会扫描所有接口
      acceptAllInterfaces = false;
    }

    // 扫描继承markerInterface的接口
    if (this.markerInterface != null) {
      addIncludeFilter(new AssignableTypeFilter(this.markerInterface) {
        // 不以类名作为匹配条件
        @Override
        protected boolean matchClassName(String className) {
          return false;
        }
      });
      // 不会扫描所有接口
      acceptAllInterfaces = false;
    }

    // 扫描所有接口
    if (acceptAllInterfaces) {
      // 包含所有类
      addIncludeFilter(new TypeFilter() {
        @Override
        public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
          return true;
        }
      });
    }

    // 排除 package-info.java
    addExcludeFilter(new TypeFilter() {
      @Override
      public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        String className = metadataReader.getClassMetadata().getClassName();
        return className.endsWith("package-info");
      }
    });
  }

  /**
   * 先调用父类扫描器获取扫描结果，然后注册所有bean的class为MapperFactoryBean，也就是说把它们定义为MapperFactoryBean
   */
  @Override
  public Set<BeanDefinitionHolder> doScan(String... basePackages) {
    // 调用父类的doScan方法
    Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);

    if (beanDefinitions.isEmpty()) {
      logger.warn("No MyBatis mapper was found in '" + Arrays.toString(basePackages) + "' package. Please check your configuration.");
    } else {
      // 处理 beanDefinitions
      processBeanDefinitions(beanDefinitions);
    }

    return beanDefinitions;
  }

  private void processBeanDefinitions(Set<BeanDefinitionHolder> beanDefinitions) {
    GenericBeanDefinition definition;
    // 遍历 beanDefinitions
    for (BeanDefinitionHolder holder : beanDefinitions) {
      definition = (GenericBeanDefinition) holder.getBeanDefinition();

      if (logger.isDebugEnabled()) {
        logger.debug("Creating MapperFactoryBean with name '" + holder.getBeanName()
          + "' and '" + definition.getBeanClassName() + "' mapperInterface");
      }

      // 获取真实接口class，并作为构造方法的参数
      definition.getConstructorArgumentValues().addGenericArgumentValue(definition.getBeanClassName());
      // 修改类为MapperFactoryBean
      definition.setBeanClass(this.mapperFactoryBean.getClass());

      // 为MapperFactoryBean的成员变量赋值

      definition.getPropertyValues().add("addToConfig", this.addToConfig);

      // 是否定义了 sqlSessionFactory，影响sqlSessionTemplate，和自动注入方式
      boolean explicitFactoryUsed = false;
      if (StringUtils.hasText(this.sqlSessionFactoryBeanName)) {
        definition.getPropertyValues().add("sqlSessionFactory", new RuntimeBeanReference(this.sqlSessionFactoryBeanName));
        explicitFactoryUsed = true;
      } else if (this.sqlSessionFactory != null) {
        definition.getPropertyValues().add("sqlSessionFactory", this.sqlSessionFactory);
        explicitFactoryUsed = true;
      }

      // 定以sqlSessionTemplate
      // 如果已定义sqlSessionFactory，则忽略sqlSessionFactory
      if (StringUtils.hasText(this.sqlSessionTemplateBeanName)) {
        if (explicitFactoryUsed) {
          logger.warn("Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.");
        }
        definition.getPropertyValues().add("sqlSessionTemplate", new RuntimeBeanReference(this.sqlSessionTemplateBeanName));
        explicitFactoryUsed = true;
      } else if (this.sqlSessionTemplate != null) {
        if (explicitFactoryUsed) {
          logger.warn("Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.");
        }
        definition.getPropertyValues().add("sqlSessionTemplate", this.sqlSessionTemplate);
        explicitFactoryUsed = true;
      }

      if (!explicitFactoryUsed) {
        if (logger.isDebugEnabled()) {
          logger.debug("Enabling autowire by type for MapperFactoryBean with name '" + holder.getBeanName() + "'.");
        }
        // 未使用sqlSessionFactory，则采用按照类型注入的方式
        definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);
      }
    }
  }
}
```

## 3. 代理

MapperFactoryBean
```java
public class MapperFactoryBean<T> extends SqlSessionDaoSupport implements FactoryBean<T> {

  private Class<T> mapperInterface;

  private boolean addToConfig = true;

  ...

  // 添加mapper，会在bean创建之后调用
  @Override
  protected void checkDaoConfig() {
    super.checkDaoConfig();

    notNull(this.mapperInterface, "Property 'mapperInterface' is required");

    Configuration configuration = getSqlSession().getConfiguration();
    if (this.addToConfig && !configuration.hasMapper(this.mapperInterface)) {
      try {
        configuration.addMapper(this.mapperInterface);
      } catch (Exception e) {
        logger.error("Error while adding the mapper '" + this.mapperInterface + "' to configuration.", e);
        throw new IllegalArgumentException(e);
      } finally {
        ErrorContext.instance().reset();
      }
    }
  }

  // 返回由MapperProxy代理的DAO对象
  @Override
  public T getObject() throws Exception {
    return getSqlSession().getMapper(this.mapperInterface);
  }
}
```

Configuration
```java
public class Configuration {

  protected final MapperRegistry mapperRegistry = new MapperRegistry(this);

  public <T> void addMapper(Class<T> type) {
    mapperRegistry.addMapper(type);
  }

  public boolean hasMapper(Class<?> type) {
    return mapperRegistry.hasMapper(type);
  }

  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
  }
}
````

MapperRegistry
```java
public class MapperRegistry {

  private final Configuration config;
  private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<Class<?>, MapperProxyFactory<?>>();

  public MapperRegistry(Configuration config) {
    this.config = config;
  }

  @SuppressWarnings("unchecked")
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
  }

  public <T> boolean hasMapper(Class<T> type) {
    return knownMappers.containsKey(type);
  }

  public <T> void addMapper(Class<T> type) {
    if (type.isInterface()) {
      if (hasMapper(type)) {
        throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
      }
      boolean loadCompleted = false;
      try {
        knownMappers.put(type, new MapperProxyFactory<T>(type));
        // It's important that the type is added before the parser is run
        // otherwise the binding may automatically be attempted by the
        // mapper parser. If the type is already known, it won't try.
        MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
        parser.parse();
        loadCompleted = true;
      } finally {
        if (!loadCompleted) {
          knownMappers.remove(type);
        }
      }
    }
  }

  /**
   * @since 3.2.2
   */
  public Collection<Class<?>> getMappers() {
    return Collections.unmodifiableCollection(knownMappers.keySet());
  }

  /**
   * @since 3.2.2
   */
  public void addMappers(String packageName, Class<?> superType) {
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
    for (Class<?> mapperClass : mapperSet) {
      addMapper(mapperClass);
    }
  }

  /**
   * @since 3.2.2
   */
  public void addMappers(String packageName) {
    addMappers(packageName, Object.class);
  }

}
```

MapperProxyFactory
```java
public class MapperProxyFactory<T> {

  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethod> methodCache = new ConcurrentHashMap<Method, MapperMethod>();

  public MapperProxyFactory(Class<T> mapperInterface) {
    this.mapperInterface = mapperInterface;
  }

  public Class<T> getMapperInterface() {
    return mapperInterface;
  }

  public Map<Method, MapperMethod> getMethodCache() {
    return methodCache;
  }

  @SuppressWarnings("unchecked")
  protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }

  public T newInstance(SqlSession sqlSession) {
    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
  }

}
```
MapperProxy
```java
public class MapperProxy<T> implements InvocationHandler, Serializable {

  private final SqlSession sqlSession;
  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethod> methodCache;

  ...

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (Object.class.equals(method.getDeclaringClass())) {
      try {
        return method.invoke(this, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    }
    final MapperMethod mapperMethod = cachedMapperMethod(method);
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

```java
public class MapperMethod {

  private final SqlCommand command;
  private final MethodSignature method;
}
```
