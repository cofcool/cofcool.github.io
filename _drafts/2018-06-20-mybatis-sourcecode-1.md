---
layout: post
category: Tech
title: Mybatis SQL执行源码分析 (一) Mapper扫描及代理
tags: [java, sourcode, mybatis]
excerpt: 本系列文章主要从源代码的角度解析Mybatis在Spirng框架上如何创建扫描，创建实例，以及SQL如何执行等核心功能。本文主要介绍Mybatis如何扫描Mapper建立接口和XML的映射关系等。
---

{% include JB/setup %}


基础环境：
* MyBatis 3.4.1
* mybatis-spring 1.3.0

Mybatis SQL执行源码分析系列文章：
* Mybatis SQL执行源码分析 (一) Mapper扫描及代理
* [Mybatis SQL执行源码分析 (二) SQL执行](./mybatis-sourcecode-2.md)


MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和Java的POJOs映射成数据库中的记录。

本系列文章主要从源代码的角度解析`Mybatis`在`Spirng`框架上如何创建扫描，创建实例，以及SQL如何执行等核心功能。本文主要介绍Mybatis如何扫描Mapper建立接口和XML的映射关系等。

目录：

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 流程](#1-流程)
	* [1.1 Mapper扫描](#11-mapper扫描)
	* [1.2 Myabtis代理DAO接口类](#12-myabtis代理dao接口类)
* [2. 源码解析](#2-源码解析)
	* [2.1 配置类](#21-配置类)
	* [2.2 扫描类](#22-扫描类)
	* [2.3 代理类](#23-代理类)

<!-- /code_chunk_output -->

## 1. 流程

### 1.1 Mapper扫描

Mybatis依靠XML文件来映射数据库和对象之间关系，配置`Mapper`如下所示（也可使用MapperScan注解），其中`basePackage`定义了需要扫描的包路径。实现`BeanDefinitionRegistryPostProcessor`接口，Spring会在项目启动时触发扫描。
```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <!-- 扫描路径 -->
  <property name="basePackage" value="net.cofcool.api.server.dao" />
  <!-- sqlSessionFactory -->
  <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
</bean>
```

我们来看看它的扫描过程。`MapperScannerConfigurer`负责读取配置和在项目启动时触发扫描过程，`ClassPathMapperScanner`负责执行扫描。

<!--
```plantuml
title: 扫描流程

MapperScannerConfigurer -> ClassPathMapperScanner: scan
ClassPathMapperScanner -> ClassPathMapperScanner: doScan
note left: 扫描配置的XML和Class，\n缓存对应关系，并把\n真实类替换为MapperFactoryBean，\n由它负责创建MapperProxy类
```
-->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="244px" preserveAspectRatio="none" style="width:416px;height:244px;" version="1.1" viewBox="0 0 416 244" width="416px" zoomAndPan="magnify"><defs><filter height="300%" id="fgbrw6jw3pf6p" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><text fill="#000000" font-family="sans-serif" font-size="14" font-weight="bold" lengthAdjust="spacingAndGlyphs" textLength="56" x="180.5" y="23.5352">扫描流程</text><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="108" x2="108" y1="68.9766" y2="203.5293"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="313" x2="313" y1="68.9766" y2="203.5293"></line><rect fill="#FEFECE" filter="url(#fgbrw6jw3pf6p)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="196" x="8" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="182" x="15" y="54.0234">MapperScannerConfigurer</text><rect fill="#FEFECE" filter="url(#fgbrw6jw3pf6p)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="196" x="8" y="202.5293"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="182" x="15" y="223.0645">MapperScannerConfigurer</text><rect fill="#FEFECE" filter="url(#fgbrw6jw3pf6p)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="187" x="218" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="173" x="225" y="54.0234">ClassPathMapperScanner</text><rect fill="#FEFECE" filter="url(#fgbrw6jw3pf6p)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="187" x="218" y="202.5293"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="173" x="225" y="223.0645">ClassPathMapperScanner</text><polygon fill="#A80036" points="301.5,95.9766,311.5,99.9766,301.5,103.9766,305.5,99.9766" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="108" x2="307.5" y1="99.9766" y2="99.9766"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="29" x="115" y="95.5449">scan</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="313.5" x2="355.5" y1="151.0635" y2="151.0635"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="355.5" x2="355.5" y1="151.0635" y2="164.0635"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="314.5" x2="355.5" y1="164.0635" y2="164.0635"></line><polygon fill="#A80036" points="324.5,160.0635,314.5,164.0635,324.5,168.0635,320.5,164.0635" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="45" x="320.5" y="146.3213">doScan</text><path d="M71,113.2871 L71,184.2871 L304,184.2871 L304,123.2871 L294,113.2871 L71,113.2871 " fill="#FBFB77" filter="url(#fgbrw6jw3pf6p)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M294,113.2871 L294,123.2871 L304,123.2871 L294,113.2871 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="151" x="77" y="130.8555">扫描配置的XML和Class，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="117" x="77" y="146.166">缓存对应关系，并把</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="212" x="77" y="161.4766">真实类替换为MapperFactoryBean，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="172" x="77" y="176.7871">由它负责创建MapperProxy类</text></g></svg>

MyBatis在扫描`Mapper`接口时，由`GenericBeanDefinition`包装DAO接口。

### 1.2 Myabtis代理DAO接口类

Mybatis通过代理的方式把定义的接口转换为可执行方法，最终实现SQL代码执行。通过`MapperFactoryBean`工厂类创建MapperProxy实例。

<!--
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
-->

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="474px" preserveAspectRatio="none" style="width:1060px;height:474px;" version="1.1" viewBox="0 0 1060 474" width="1060px" zoomAndPan="magnify"><defs><filter height="300%" id="fa73bahuo6npo" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><text fill="#000000" font-family="sans-serif" font-size="14" font-weight="bold" lengthAdjust="spacingAndGlyphs" textLength="224" x="418.75" y="23.5352">MapperProxy代理DAO接口类过程</text><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="302" x2="302" y1="68.9766" y2="434.3242"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="434" x2="434" y1="68.9766" y2="434.3242"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="573.5" x2="573.5" y1="68.9766" y2="434.3242"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="702.5" x2="702.5" y1="68.9766" y2="434.3242"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="854.5" x2="854.5" y1="68.9766" y2="434.3242"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="998.5" x2="998.5" y1="68.9766" y2="434.3242"></line><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="149" x="226" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="135" x="233" y="54.0234">MapperFactoryBean</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="149" x="226" y="433.3242"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="135" x="233" y="453.8594">MapperFactoryBean</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="87" x="389" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="73" x="396" y="54.0234">SqlSession</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="87" x="389" y="433.3242"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="73" x="396" y="453.8594">SqlSession</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="110" x="516.5" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="96" x="523.5" y="54.0234">Configuration</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="110" x="516.5" y="433.3242"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="96" x="523.5" y="453.8594">Configuration</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="121" x="640.5" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="107" x="647.5" y="54.0234">MapperRegistry</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="121" x="640.5" y="433.3242"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="107" x="647.5" y="453.8594">MapperRegistry</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="155" x="775.5" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="141" x="782.5" y="54.0234">MapperProxyFactory</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="155" x="775.5" y="433.3242"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="141" x="782.5" y="453.8594">MapperProxyFactory</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="105" x="944.5" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="91" x="951.5" y="54.0234">MapperProxy</text><rect fill="#FEFECE" filter="url(#fa73bahuo6npo)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="105" x="944.5" y="433.3242"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="91" x="951.5" y="453.8594">MapperProxy</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="302.5" x2="344.5" y1="100.2871" y2="100.2871"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="344.5" x2="344.5" y1="100.2871" y2="113.2871"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="303.5" x2="344.5" y1="113.2871" y2="113.2871"></line><polygon fill="#A80036" points="313.5,109.2871,303.5,113.2871,313.5,117.2871,309.5,113.2871" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="69" x="309.5" y="95.5449">getObject()</text><polygon fill="#A80036" points="422.5,189.2188,432.5,193.2188,422.5,197.2188,426.5,193.2188" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="302.5" x2="428.5" y1="193.2188" y2="193.2188"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="95" x="309.5" y="188.7871">getSqlSession()</text><path d="M8,126.2871 L8,243.2871 L294,243.2871 L294,136.2871 L284,126.2871 L8,126.2871 " fill="#FBFB77" filter="url(#fa73bahuo6npo)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M284,126.2871 L284,136.2871 L294,136.2871 L284,126.2871 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="195" x="14" y="143.8555">ClassPathMapperScanner在创建</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="147" x="14" y="159.166">MapperFactoryBean时，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="143" x="14" y="174.4766">会把sqlSessionFactory,</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="125" x="14" y="189.7871">sqlSessionTemplate</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="212" x="14" y="205.0977">等参数传递给MapperFactoryBean，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="265" x="14" y="220.4082">从而创建SqlSession(SqlSessionTemplate)，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="152" x="14" y="235.7188">通过getObject返回代理类</text><polygon fill="#A80036" points="561.5,269.4609,571.5,273.4609,561.5,277.4609,565.5,273.4609" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="434.5" x2="567.5" y1="273.4609" y2="273.4609"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="115" x="441.5" y="269.0293">getConfiguration()</text><polygon fill="#A80036" points="691,298.7715,701,302.7715,691,306.7715,695,302.7715" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="573.5" x2="697" y1="302.7715" y2="302.7715"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="74" x="580.5" y="298.3398">getMapper()</text><polygon fill="#A80036" points="843,348.3926,853,352.3926,843,356.3926,847,352.3926" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="703" x2="849" y1="352.3926" y2="352.3926"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="74" x="710" y="347.9609">getMapper()</text><path d="M478,316.082 L478,371.082 L694,371.082 L694,326.082 L684,316.082 L478,316.082 " fill="#FBFB77" filter="url(#fa73bahuo6npo)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M684,316.082 L684,326.082 L694,326.082 L684,316.082 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="124" x="484" y="333.6504">调用mapperRegistry</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="189" x="484" y="348.9609">的getMapper(type, sqlSession)</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="195" x="484" y="364.2715">，并由代理工厂类创建代理类实例</text><polygon fill="#A80036" points="987,398.0137,997,402.0137,987,406.0137,991,402.0137" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="855" x2="993" y1="402.0137" y2="402.0137"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="86" x="862" y="397.582">newInstance()</text><polygon fill="#A80036" points="313.5,412.3242,303.5,416.3242,313.5,420.3242,309.5,416.3242" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="307.5" x2="998" y1="416.3242" y2="416.3242"></line></g></svg>

## 2. 源码解析

### 2.1 配置类

**MapperScannerConfigurer**，根据`basePackage`，调用`ClassPathMapperScanner`的扫描方法进行扫描，可配置扫描路径，扫描条件，包括注解，父类等。

```java
public class MapperScannerConfigurer implements BeanDefinitionRegistryPostProcessor, InitializingBean, ApplicationContextAware, BeanNameAware {

  // 扫描包路径，如果有多个路径，可用,或;等字符分割
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

### 2.2 扫描类

**ClassPathMapperScanner**，执行具体的扫描，继承Spring的`ClassPathBeanDefinitionScanner`类，ClassPathBeanDefinitionScanner是Spring提供的一个用于扫描Bean定义配置的基础类，ClassPathMapperScanner在其基础上配置了扫描类的过滤条件和类定义替换等。

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
      // addToConfig
      definition.getPropertyValues().add("addToConfig", this.addToConfig);

      // sqlSession
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
        // 采用按照类型注入的方式
        definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);
      }
    }
  }
}
```

### 2.3 代理类

**MapperFactoryBean**，该类会代理项目定义的DAO接口，由于实现了`FactoryBean`接口，通过调用`getObject`方法获取实例，也就是MapperProxy对象。

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

    // 通过Configuration来添加Mapper
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
    // 调用父类的 getSqlSession() 获取SqlSession
    return getSqlSession().getMapper(this.mapperInterface);
  }
}
```

MapperFactoryBean的父类为`SqlSessionDaoSupport`，存储了sqlSession对象。

```java
public abstract class SqlSessionDaoSupport extends DaoSupport {

  private SqlSession sqlSession;

  private boolean externalSqlSession;

  // 上文提到ClassPathMapperScanner通过GenericBeanDefinition给MapperFactoryBean的sqlSessionFactory，sqlSessionTemplate属性赋值，也就是调用它们的setter方法
  // 在setter中给sqlSession进行赋值
  public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
    if (!this.externalSqlSession) {
      this.sqlSession = new SqlSessionTemplate(sqlSessionFactory);
    }
  }

  public void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {
    this.sqlSession = sqlSessionTemplate;
    this.externalSqlSession = true;
  }

  public SqlSession getSqlSession() {
    return this.sqlSession;
  }

  // 检查属性，sqlSessionFactory或sqlSessionTemplate至少存在一个
  // 具体可查看ClassPathMapperScanner的processBeanDefinitions方法
  @Override
  protected void checkDaoConfig() {
    notNull(this.sqlSession, "Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required");
  }

}
```

上文提到，`Configuration`类存储和管理Mapper，它通过`MapperRegistry`来添加和存储Mapper。

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

  ...
}
```

**MapperRegistry**，管理Mapper的具体创建，通过`MapperProxyFactory`来创建`MapperProxy`实例。

```java
public class MapperRegistry {

  private final Configuration config;

  // 缓存MapperProxyFactory和MapperInterface的关系，MapperInterface为key
  private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<Class<?>, MapperProxyFactory<?>>();

  public MapperRegistry(Configuration config) {
    this.config = config;
  }

  // 通过MapperInterface获取MapperProxy实例
  @SuppressWarnings("unchecked")
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    // 从缓存中读取MapperProxyFactory
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      // 由MapperProxyFactory创建MapperProxy实例
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
  }

  // 添加Mapper
  public <T> void addMapper(Class<T> type) {
    // 是否是接口类型
    if (type.isInterface()) {
      // 是否已存在
      if (hasMapper(type)) {
        throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
      }
      boolean loadCompleted = false;
      try {
        // 缓存
        knownMappers.put(type, new MapperProxyFactory<T>(type));
        // 方法解析，包括方法定义合法性检查，接口与XML映射，注解解析，缓存设置等，更多可查看MapperAnnotationBuilder类
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

  ...

}
```

**MapperProxyFactory**，MapperProxy的工厂方法，负责创建MapperProxy实例。
```java
public class MapperProxyFactory<T> {

  // 定义的接口
  private final Class<T> mapperInterface;
  // 方法和sql执行缓存
  // MapperMethod封装了SqlSession的执行
  private final Map<Method, MapperMethod> methodCache = new ConcurrentHashMap<Method, MapperMethod>();

  public MapperProxyFactory(Class<T> mapperInterface) {
    this.mapperInterface = mapperInterface;
  }

  ...

  // 由MapperProxy代理定义的接口
  @SuppressWarnings("unchecked")
  protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }

  // 根据SqlSession创建MapperProxy实例
  public T newInstance(SqlSession sqlSession) {
    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
  }

}
```

通过一系列的调用，最终创建了MapperProxy实例。该类负责把我们定义的DAO方法以代理的方式转换为可执行的SQL代码。
