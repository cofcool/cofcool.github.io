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

* [Mybatis SQL执行源码分析 (一) Mapper扫描及代理](./2018-06-20-mybatis-sourcecode-1.md)
* Mybatis SQL执行源码分析 (二) SQL执行


MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和Java的POJOs映射成数据库中的记录。

本系列文章主要从源代码的角度解析`Mybatis`在`Spirng`框架上如何创建扫描，创建实例，以及SQL如何执行等核心功能。本文主要介绍SQL执行过程。

目录：

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=4 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 流程](#1-流程)
	* [1.1 SqlSession创建流程](#11-sqlsession创建流程)
	* [1.2 DAO执行流程](#12-dao执行流程)
* [2. 代码分析](#2-代码分析)
	* [2.1 创建 SqlSession](#21-创建-sqlsession)
	* [2.2 SQL执行](#22-sql执行)
		* [2.2.1 SqlSession简要介绍](#221-sqlsession简要介绍)
		* [2.2.2 执行](#222-执行)
	* [2.3 事务处理](#23-事务处理)
		* [2.3.1 Mybatis管理](#231-mybatis管理)
		* [2.3.2 Spring管理](#232-spring管理)
	* [2.4 插件](#24-插件)
* [3. 缓存](#3-缓存)

<!-- /code_chunk_output -->

## 1. 流程

### 1.1 SqlSession创建流程

首先看看SqlSession的创建流程。

<!--
```plantuml
title: 创建SqlSessionFactory实例
SqlSessionFactoryBean -> SqlSessionFactoryBean: getObject
note left: 实现FactoryBean接口，\n根据Configuration来\nbuildSqlSessionFactory，\n包括事务，DataSource等
SqlSessionFactoryBean -> SqlSessionFactoryBuilder: afterPropertiesSet
SqlSessionFactory <- SqlSessionFactoryBuilder: build
SqlSessionFactory -> SqlSessionFactoryBean

```
-->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="287px" preserveAspectRatio="none" style="width:640px;height:287px;" version="1.1" viewBox="0 0 640 287" width="640px" zoomAndPan="magnify"><defs><filter height="300%" id="fdpi1kqxc8h4u" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><text fill="#000000" font-family="sans-serif" font-size="14" font-weight="bold" lengthAdjust="spacingAndGlyphs" textLength="185" x="228" y="23.5352">创建SqlSessionFactory实例</text><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="196" x2="196" y1="68.9766" y2="246.8398"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="387" x2="387" y1="68.9766" y2="246.8398"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="562" x2="562" y1="68.9766" y2="246.8398"></line><rect fill="#FEFECE" filter="url(#fdpi1kqxc8h4u)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="170" x="109" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="156" x="116" y="54.0234">SqlSessionFactoryBean</text><rect fill="#FEFECE" filter="url(#fdpi1kqxc8h4u)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="170" x="109" y="245.8398"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="156" x="116" y="266.375">SqlSessionFactoryBean</text><rect fill="#FEFECE" filter="url(#fdpi1kqxc8h4u)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="185" x="293" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="171" x="300" y="54.0234">SqlSessionFactoryBuilder</text><rect fill="#FEFECE" filter="url(#fdpi1kqxc8h4u)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="185" x="293" y="245.8398"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="171" x="300" y="266.375">SqlSessionFactoryBuilder</text><rect fill="#FEFECE" filter="url(#fdpi1kqxc8h4u)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="137" x="492" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="123" x="499" y="54.0234">SqlSessionFactory</text><rect fill="#FEFECE" filter="url(#fdpi1kqxc8h4u)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="137" x="492" y="245.8398"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="123" x="499" y="266.375">SqlSessionFactory</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="196" x2="238" y1="121.7529" y2="121.7529"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="238" x2="238" y1="121.7529" y2="134.7529"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="197" x2="238" y1="134.7529" y2="134.7529"></line><polygon fill="#A80036" points="207,130.7529,197,134.7529,207,138.7529,203,134.7529" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="61" x="203" y="117.0107">getObject</text><path d="M8,83.9766 L8,154.9766 L187,154.9766 L187,93.9766 L177,83.9766 L8,83.9766 " fill="#FBFB77" filter="url(#fdpi1kqxc8h4u)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M177,83.9766 L177,93.9766 L187,93.9766 L177,83.9766 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="140" x="14" y="101.5449">实现FactoryBean接口，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="126" x="14" y="116.8555">根据Configuration来</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="158" x="14" y="132.166">buildSqlSessionFactory，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="149" x="14" y="147.4766">包括事务，DataSource等</text><polygon fill="#A80036" points="375.5,181.2188,385.5,185.2188,375.5,189.2188,379.5,185.2188" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="196" x2="381.5" y1="185.2188" y2="185.2188"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="111" x="203" y="180.7871">afterPropertiesSet</text><polygon fill="#A80036" points="550.5,210.5293,560.5,214.5293,550.5,218.5293,554.5,214.5293" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="387.5" x2="556.5" y1="214.5293" y2="214.5293"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="32" x="394.5" y="210.0977">build</text><polygon fill="#A80036" points="207,224.8398,197,228.8398,207,232.8398,203,228.8398" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="201" x2="561.5" y1="228.8398" y2="228.8398"></line>></g></svg>

<!--
```plantuml
title: 创建SqlSession实例
DefaultSqlSessionFactory -> DefaultSqlSessionFactory: openSession
note left: 默认实现为\nDefaultSqlSessionFactory，\n根据Configuration创建事务，\nExecutor等，并以此创建\nDefaultSqlSession实例
DefaultSqlSessionFactory -> DefaultSqlSession: openSessionFromDataSource/\nopenSessionFromConnection
DefaultSqlSession -> DefaultSqlSessionFactory

```
-->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="288px" preserveAspectRatio="none" style="width:506px;height:288px;" version="1.1" viewBox="0 0 506 288" width="506px" zoomAndPan="magnify"><defs><filter height="300%" id="fojg5y2xotfx0" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><text fill="#000000" font-family="sans-serif" font-size="14" font-weight="bold" lengthAdjust="spacingAndGlyphs" textLength="133" x="187" y="23.5352">创建SqlSession实例</text><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="216" x2="216" y1="68.9766" y2="248.1504"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="429" x2="429" y1="68.9766" y2="248.1504"></line><rect fill="#FEFECE" filter="url(#fojg5y2xotfx0)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="186" x="121" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="172" x="128" y="54.0234">DefaultSqlSessionFactory</text><rect fill="#FEFECE" filter="url(#fojg5y2xotfx0)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="186" x="121" y="247.1504"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="172" x="128" y="267.6855">DefaultSqlSessionFactory</text><rect fill="#FEFECE" filter="url(#fojg5y2xotfx0)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="136" x="359" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="122" x="366" y="54.0234">DefaultSqlSession</text><rect fill="#FEFECE" filter="url(#fojg5y2xotfx0)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="136" x="359" y="247.1504"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="122" x="366" y="267.6855">DefaultSqlSession</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="216" x2="258" y1="129.4082" y2="129.4082"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="258" x2="258" y1="129.4082" y2="142.4082"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="217" x2="258" y1="142.4082" y2="142.4082"></line><polygon fill="#A80036" points="227,138.4082,217,142.4082,227,146.4082,223,142.4082" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="79" x="223" y="124.666">openSession</text><path d="M8,83.9766 L8,169.9766 L207,169.9766 L207,93.9766 L197,83.9766 L8,83.9766 " fill="#FBFB77" filter="url(#fojg5y2xotfx0)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M197,83.9766 L197,93.9766 L207,93.9766 L197,83.9766 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="65" x="14" y="101.5449">默认实现为</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="172" x="14" y="116.8555">DefaultSqlSessionFactory，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="178" x="14" y="132.166">根据Configuration创建事务，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="146" x="14" y="147.4766">Executor等，并以此创建</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="139" x="14" y="162.7871">DefaultSqlSession实例</text><polygon fill="#A80036" points="417,211.5293,427,215.5293,417,219.5293,421,215.5293" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="216" x2="423" y1="215.5293" y2="215.5293"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="189" x="223" y="196.0977">openSessionFromDataSource/</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="183" x="223" y="211.4082">openSessionFromConnection</text><polygon fill="#A80036" points="227,226.1504,217,230.1504,227,234.1504,223,230.1504" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="221" x2="428" y1="230.1504" y2="230.1504"></line></g></svg>

### 1.2 DAO执行流程

如下图所示:

<!--
```plantuml
title: DAO执行流程

UserMapper -> MapperProxy: 查询用户: select
MapperProxy -> MapperMethod: invoke
note left: 代理扫描到的DAO接口
MapperMethod -> SqlSessionTemplate: execute
note left: 以执行query为例
SqlSessionTemplate -> this.sqlSessionProxy: select
note left: SqlSessionTemplate的\nsqlSessionProxy持有\nSqlSession接口的代理
this.sqlSessionProxy -> SqlSession: invoke
note left: 具体可查看SqlSessionTemplate的\n内部类SqlSessionInterceptor
SqlSession -> Executor: select
Executor -> Executor: query
note left: Executor负责具体的SQL执行，\n包含SIMPLE, REUSE, BATCH三种
Executor -> UserMapper
```
-->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="459px" preserveAspectRatio="none" style="width:903px;height:459px;" version="1.1" viewBox="0 0 903 459" width="903px" zoomAndPan="magnify"><defs><filter height="300%" id="fzfel5prqxvs5" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><text fill="#000000" font-family="sans-serif" font-size="14" font-weight="bold" lengthAdjust="spacingAndGlyphs" textLength="89" x="406.25" y="23.5352">DAO执行流程</text><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="58" x2="58" y1="68.9766" y2="419.3926"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="179" x2="179" y1="68.9766" y2="419.3926"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="305" x2="305" y1="68.9766" y2="419.3926"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="453" x2="453" y1="68.9766" y2="419.3926"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="620" x2="620" y1="68.9766" y2="419.3926"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="754" x2="754" y1="68.9766" y2="419.3926"></line><line style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 5.0,5.0;" x1="849" x2="849" y1="68.9766" y2="419.3926"></line><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="97" x="8" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="83" x="15" y="54.0234">UserMapper</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="97" x="8" y="418.3926"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="83" x="15" y="438.9277">UserMapper</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="105" x="125" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="91" x="132" y="54.0234">MapperProxy</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="105" x="125" y="418.3926"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="91" x="132" y="438.9277">MapperProxy</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="118" x="244" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="104" x="251" y="54.0234">MapperMethod</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="118" x="244" y="418.3926"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="104" x="251" y="438.9277">MapperMethod</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="151" x="376" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="137" x="383" y="54.0234">SqlSessionTemplate</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="151" x="376" y="418.3926"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="137" x="383" y="438.9277">SqlSessionTemplate</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="154" x="541" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="140" x="548" y="54.0234">this.sqlSessionProxy</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="154" x="541" y="418.3926"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="140" x="548" y="438.9277">this.sqlSessionProxy</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="87" x="709" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="73" x="716" y="54.0234">SqlSession</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="87" x="709" y="418.3926"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="73" x="716" y="438.9277">SqlSession</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="75" x="810" y="33.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="61" x="817" y="54.0234">Executor</text><rect fill="#FEFECE" filter="url(#fzfel5prqxvs5)" height="30.4883" style="stroke: #A80036; stroke-width: 1.5;" width="75" x="810" y="418.3926"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="61" x="817" y="438.9277">Executor</text><polygon fill="#A80036" points="167.5,95.9766,177.5,99.9766,167.5,103.9766,171.5,99.9766" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="58.5" x2="173.5" y1="99.9766" y2="99.9766"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="97" x="65.5" y="95.5449">查询用户: select</text><polygon fill="#A80036" points="293,130.2871,303,134.2871,293,138.2871,297,134.2871" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="179.5" x2="299" y1="134.2871" y2="134.2871"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="42" x="186.5" y="129.8555">invoke</text><path d="M16,113.2871 L16,138.2871 L170,138.2871 L170,123.2871 L160,113.2871 L16,113.2871 " fill="#FBFB77" filter="url(#fzfel5prqxvs5)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M160,113.2871 L160,123.2871 L170,123.2871 L160,113.2871 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="133" x="22" y="130.8555">代理扫描到的DAO接口</text><polygon fill="#A80036" points="441.5,169.5977,451.5,173.5977,441.5,177.5977,445.5,173.5977" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="305" x2="447.5" y1="173.5977" y2="173.5977"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="49" x="312" y="169.166">execute</text><path d="M175,152.5977 L175,177.5977 L296,177.5977 L296,162.5977 L286,152.5977 L175,152.5977 " fill="#FBFB77" filter="url(#fzfel5prqxvs5)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M286,152.5977 L286,162.5977 L296,162.5977 L286,152.5977 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="100" x="181" y="170.166">以执行query为例</text><polygon fill="#A80036" points="608,224.2188,618,228.2188,608,232.2188,612,228.2188" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="453.5" x2="614" y1="228.2188" y2="228.2188"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="37" x="460.5" y="223.7871">select</text><path d="M285,191.9082 L285,246.9082 L444,246.9082 L444,201.9082 L434,191.9082 L285,191.9082 " fill="#FBFB77" filter="url(#fzfel5prqxvs5)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M434,191.9082 L434,201.9082 L444,201.9082 L434,191.9082 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="138" x="291" y="209.4766">SqlSessionTemplate的</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="128" x="291" y="224.7871">sqlSessionProxy持有</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="132" x="291" y="240.0977">SqlSession接口的代理</text><polygon fill="#A80036" points="742.5,286.4951,752.5,290.4951,742.5,294.4951,746.5,290.4951" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="620" x2="748.5" y1="290.4951" y2="290.4951"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="42" x="627" y="286.0635">invoke</text><path d="M387,261.8398 L387,301.8398 L611,301.8398 L611,271.8398 L601,261.8398 L387,261.8398 " fill="#FBFB77" filter="url(#fzfel5prqxvs5)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M601,261.8398 L601,271.8398 L611,271.8398 L601,261.8398 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="203" x="393" y="279.4082">具体可查看SqlSessionTemplate的</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="175" x="393" y="294.7188">内部类SqlSessionInterceptor</text><polygon fill="#A80036" points="837.5,328.4609,847.5,332.4609,837.5,336.4609,841.5,332.4609" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="754.5" x2="843.5" y1="332.4609" y2="332.4609"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="37" x="761.5" y="328.0293">select</text><line style="stroke: #A80036; stroke-width: 1.0;" x1="849.5" x2="891.5" y1="368.2373" y2="368.2373"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="891.5" x2="891.5" y1="368.2373" y2="381.2373"></line><line style="stroke: #A80036; stroke-width: 1.0;" x1="850.5" x2="891.5" y1="381.2373" y2="381.2373"></line><polygon fill="#A80036" points="860.5,377.2373,850.5,381.2373,860.5,385.2373,856.5,381.2373" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="35" x="856.5" y="363.4951">query</text><path d="M627,345.7715 L627,385.7715 L840,385.7715 L840,355.7715 L830,345.7715 L627,345.7715 " fill="#FBFB77" filter="url(#fzfel5prqxvs5)" style="stroke: #A80036; stroke-width: 1.0;"></path><path d="M830,345.7715 L830,355.7715 L840,355.7715 L830,345.7715 " fill="#FBFB77" style="stroke: #A80036; stroke-width: 1.0;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="183" x="633" y="363.3398">Executor负责具体的SQL执行，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="192" x="633" y="378.6504">包含SIMPLE, REUSE, BATCH三种</text><polygon fill="#A80036" points="69.5,397.3926,59.5,401.3926,69.5,405.3926,65.5,401.3926" style="stroke: #A80036; stroke-width: 1.0;"></polygon><line style="stroke: #A80036; stroke-width: 1.0;" x1="63.5" x2="848.5" y1="401.3926" y2="401.3926"></line></g></svg>

## 2. 代码分析

经过以上分析可了解到Myabtis的基本执行流程和相关类，下面我们来看具体的代码实现。

### 2.1 创建 SqlSession

**SqlSessionFactoryBean**，Spring容器调用它来创建SqlSessionFactory。

```java
// 创建 SqlSessionFactory
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

SqlSessionFactory的默认实现`DefaultSqlSessionFactory`，openSession方法调用`openSessionFromDataSource`或`openSessionFromConnection`函数来创建SqlSession实例。
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

#### 2.2.1 SqlSession简要介绍

在Mybatis中，DAO操作依靠SqlSession来执行。独立使用时，sqlSession实例为`DefaultSqlSession`，需要手动管理该实例，以及事务处理等；若使用了mybatis-spring，则为`SqlSessionTemplate`，它的创建以及管理等由Spring负责。

##### 1. 独立使用

Mybatis提供了`SqlSessionManager`，可由它来创建和管理SqlSession，也可直接通过`SqlSessionFactory`来创建SqlSession实例。

SqlSessionManager代理了SqlSession，相关操作借由SqlSessionManager的私有类SqlSessionInterceptor完成。

```java
private class SqlSessionInterceptor implements InvocationHandler {
  ...

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    // 从线程中获取SqlSession
    final SqlSession sqlSession = SqlSessionManager.this.localSqlSession.get();
    if (sqlSession != null) {
      try {
        // 若不为NULL则直接调用
        // 未包含事务等操作
        return method.invoke(sqlSession, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    } else {
      // 未从线程中取到
      // 创建SqlSession
      // 自动提交事务，回滚以及关闭
      final SqlSession autoSqlSession = openSession();
      try {
        final Object result = method.invoke(autoSqlSession, args);
        autoSqlSession.commit();
        return result;
      } catch (Throwable t) {
        autoSqlSession.rollback();
        throw ExceptionUtil.unwrapThrowable(t);
      } finally {
        autoSqlSession.close();
      }
    }
  }
}
```

##### 2. Spring容器中运行

在上一篇文章介绍到MapperFactoryBean会代理定义的DAO接口，并把 `SqlSessionTemplate`赋值给`this.sqlSession`。SqlSessionTemplate实现SqlSession接口，并持有SqlSessionFactory实例，是一个特殊的SqlSession实例，主要功能为把Spring容器中的操作转换为Mybaits中SqlSession对应的操作。

```java
public class SqlSessionTemplate implements SqlSession, DisposableBean {

  public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType,
        PersistenceExceptionTranslator exceptionTranslator) {

  notNull(sqlSessionFactory, "Property 'sqlSessionFactory' is required");
  notNull(executorType, "Property 'executorType' is required");

  this.sqlSessionFactory = sqlSessionFactory;
  this.executorType = executorType;
  this.exceptionTranslator = exceptionTranslator;

  // 创建代理
  this.sqlSessionProxy = (SqlSession) newProxyInstance(
    SqlSessionFactory.class.getClassLoader(),
    new Class[] { SqlSession.class },
    new SqlSessionInterceptor());
  }
  ...
}
```

当执行DAO方法时，代理类经过一系列调用会调用到SqlSessionTemplate的系列方法，SqlSessionTemplate的sqlSessionProxy代理了SqlSession的实例，在调用这些方法时实际会调用`SqlSessionTemplate`的内部类`SqlSessionInterceptor`的invoke方法。

```java
private class SqlSessionInterceptor implements InvocationHandler {
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    // 获取 SqlSession，通过Spring的TransactionSynchronizationManager管理
    SqlSession sqlSession = getSqlSession(
        SqlSessionTemplate.this.sqlSessionFactory,
        SqlSessionTemplate.this.executorType,
        SqlSessionTemplate.this.exceptionTranslator);
    try {
      // 调用SqlSession的方法
      Object result = method.invoke(sqlSession, args);
      if (!isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
        // 提交事务
        sqlSession.commit(true);
      }
      return result;
    } catch (Throwable t) {
      Throwable unwrapped = unwrapThrowable(t);
      if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {
        // 抛出异常时关闭sqlSession，并置为NULL
        closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
        sqlSession = null;
        // 异常转换，转为Spring定义的数据库访问异常
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

#### 2.2.2 执行

由“MapperProxy代理DAO接口类过程”可知，`MapperProxy`为真正的DAO接口实例，该类持有sqlSession，mapperInterface，其中mapperInterface为定义的DAO接口类。

```java
public class MapperProxy<T> implements InvocationHandler, Serializable {

  private final SqlSession sqlSession;
  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethod> methodCache;

  ...

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    // 如果是Object的方法，则直接调用
    if (Object.class.equals(method.getDeclaringClass())) {
      try {
        return method.invoke(this, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    }
    // 执行代理方法
    // MapperMethod封装了MethodSignature(方法参数，返回值等)和SqlCommand(Sql类型，方法ID)
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    // 通过sqlSession执行sql
    return mapperMethod.execute(sqlSession, args);
  }

  // 缓存Method，MapperMethod
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

MapperMethod封装了MethodSignature(方法参数，返回值等)和SqlCommand(Sql类型，方法ID)，把接口定义方法转换为对应的SqlSession方法，从而执行SQL语句。

```java
public class MapperMethod {

  // 封装方法名和类型
  private final SqlCommand command;
  // 封装方法参数，返回值等
  private final MethodSignature method;

  public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
    // Configuration封装了MappedStatement，根据传入的mapperInterface获取对应的SqlCommand，MethodSignature
    this.command = new SqlCommand(config, mapperInterface, method);
    this.method = new MethodSignature(config, mapperInterface, method);
  }

  // 执行
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
            // 含有参数类型为ResultHandler且没有返回值
            executeWithResultHandler(sqlSession, args);
            result = null;
          } else if (method.returnsMany()) {
            // set或array
            result = executeForMany(sqlSession, args);
          } else if (method.returnsMap()) {
            // 返回值类型为Map，且有注解MapKey（定义key）
            // 把返回结果以key-value的形式存储
            result = executeForMap(sqlSession, args);
          } else if (method.returnsCursor()) {
            // 返回值类型为Cursor
            result = executeForCursor(sqlSession, args);
          } else {
            // 方法有返回值
            Object param = method.convertArgsToSqlCommandParam(args);
            result = sqlSession.selectOne(command.getName(), param);
          }
          break;
        case FLUSH:
          // 处理批量执行结果
          result = sqlSession.flushStatements();
          break;
        default:
          throw new BindingException("Unknown execution method for: " + command.getName());
      }
      ...
      return result;
    }

    ...
}
```

经过`MapperMethod`处理之后，SqlSession开始登场，定义的DAO方法通过SqlSession开始执行。

SqlSession的默认实现为`DefaultSqlSession`，它的构造方法为`public DefaultSqlSession(Configuration configuration, Executor executor, boolean autoCommit)`，在创建实例的时候传入`Configuration`。DefaultSqlSession在执行Sql时需要获取配置的动态sql以及相关参数等，`MappedStatement`封装了这些数据，可通过Configuration获取`MappedStatement`实例。

以DefaultSqlSession的selectList为例：
```java
@Override
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
  try {
    // MappedStatement
    MappedStatement ms = configuration.getMappedStatement(statement);
    // executor执行查询
    return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

MappedStatement,该类封装了Mapper定义的方法执行时需要的各种参数定义和配置。

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
  // FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认未设置
  // FORWARD_ONLY: Cursor只能向前迭代
  // SCROLL_SENSITIVE：Cursor前后迭代都可以，对ResultSet创建之后数据库数据发生改变不敏感
  // SCROLL_INSENSITIVE：Cursor前后迭代都可以，对ResultSet创建之后数据库数据发生改变敏感
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
  // 这个设置仅对多结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称是逗号分隔的。
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

  // BoundSql封装了xml或注解中定义的动态sql
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

**Executor**负责执行具体的sql，定义了CURD等各种方法。

```java
public interface Executor {

  ...

  int update(MappedStatement ms, Object parameter) throws SQLException;

  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey cacheKey, BoundSql boundSql) throws SQLException;

  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;

  <E> Cursor<E> queryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds) throws SQLException;

  List<BatchResult> flushStatements() throws SQLException;

  void commit(boolean required) throws SQLException;

  void rollback(boolean required) throws SQLException;

  void close(boolean forceRollback);

  boolean isClosed();

  ...

}
```

Configuration创建Executor。

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
```

类图如下所示:

SqlSession

<!--
```plantuml
interface SqlSession

SqlSession <|.. DefaultSqlSession
SqlSession <|.. SqlSessionTemplate
SqlSessionTemplate --/> sqlSessionProxy: SqlSession
SqlSessionTemplate --/> SqlSessionFactory
```
-->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="301px" preserveAspectRatio="none" style="width:404px;height:301px;" version="1.1" viewBox="0 0 404 301" width="404px" zoomAndPan="magnify"><defs><filter height="300%" id="f1vwfwq1xir8hg" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><!--class SqlSession--><rect fill="#FEFECE" filter="url(#f1vwfwq1xir8hg)" height="48" id="SqlSession" style="stroke: #A80036; stroke-width: 1.5;" width="91" x="46.5" y="8"></rect><ellipse cx="61.5" cy="24" fill="#B4A7E5" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M57.4277,19.7651 L57.4277,17.6069 L64.8071,17.6069 L64.8071,19.7651 L62.3418,19.7651 L62.3418,27.8418 L64.8071,27.8418 L64.8071,30 L57.4277,30 L57.4277,27.8418 L59.8931,27.8418 L59.8931,19.7651 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" font-style="italic" lengthAdjust="spacingAndGlyphs" textLength="59" x="75.5" y="28.5352">SqlSession</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="47.5" x2="136.5" y1="40" y2="40"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="47.5" x2="136.5" y1="48" y2="48"></line><!--class DefaultSqlSession--><rect fill="#FEFECE" filter="url(#f1vwfwq1xir8hg)" height="48" id="DefaultSqlSession" style="stroke: #A80036; stroke-width: 1.5;" width="132" x="6" y="117"></rect><ellipse cx="21" cy="133" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M23.9731,138.6431 Q23.3921,138.9419 22.7529,139.0913 Q22.1138,139.2407 21.4082,139.2407 Q18.9014,139.2407 17.5815,137.5889 Q16.2617,135.937 16.2617,132.8159 Q16.2617,129.6865 17.5815,128.0347 Q18.9014,126.3828 21.4082,126.3828 Q22.1138,126.3828 22.7612,126.5322 Q23.4087,126.6816 23.9731,126.9805 L23.9731,129.7031 Q23.3423,129.1221 22.7488,128.8523 Q22.1553,128.5825 21.5244,128.5825 Q20.1797,128.5825 19.4949,129.6492 Q18.8101,130.7158 18.8101,132.8159 Q18.8101,134.9077 19.4949,135.9744 Q20.1797,137.041 21.5244,137.041 Q22.1553,137.041 22.7488,136.7712 Q23.3423,136.5015 23.9731,135.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="100" x="35" y="137.5352">DefaultSqlSession</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="7" x2="137" y1="149" y2="149"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="7" x2="137" y1="157" y2="157"></line><!--class SqlSessionTemplate--><rect fill="#FEFECE" filter="url(#f1vwfwq1xir8hg)" height="48" id="SqlSessionTemplate" style="stroke: #A80036; stroke-width: 1.5;" width="146" x="173" y="117"></rect><ellipse cx="188" cy="133" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M190.9731,138.6431 Q190.3921,138.9419 189.7529,139.0913 Q189.1138,139.2407 188.4082,139.2407 Q185.9014,139.2407 184.5815,137.5889 Q183.2617,135.937 183.2617,132.8159 Q183.2617,129.6865 184.5815,128.0347 Q185.9014,126.3828 188.4082,126.3828 Q189.1138,126.3828 189.7612,126.5322 Q190.4087,126.6816 190.9731,126.9805 L190.9731,129.7031 Q190.3423,129.1221 189.7488,128.8523 Q189.1553,128.5825 188.5244,128.5825 Q187.1797,128.5825 186.4949,129.6492 Q185.8101,130.7158 185.8101,132.8159 Q185.8101,134.9077 186.4949,135.9744 Q187.1797,137.041 188.5244,137.041 Q189.1553,137.041 189.7488,136.7712 Q190.3423,136.5015 190.9731,135.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="114" x="202" y="137.5352">SqlSessionTemplate</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="174" x2="318" y1="149" y2="149"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="174" x2="318" y1="157" y2="157"></line><!--class sqlSessionProxy--><rect fill="#FEFECE" filter="url(#f1vwfwq1xir8hg)" height="48" id="sqlSessionProxy" style="stroke: #A80036; stroke-width: 1.5;" width="123" x="102.5" y="242"></rect><ellipse cx="117.5" cy="258" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M120.4731,263.6431 Q119.8921,263.9419 119.2529,264.0913 Q118.6138,264.2407 117.9082,264.2407 Q115.4014,264.2407 114.0815,262.5889 Q112.7617,260.937 112.7617,257.8159 Q112.7617,254.6865 114.0815,253.0347 Q115.4014,251.3828 117.9082,251.3828 Q118.6138,251.3828 119.2612,251.5322 Q119.9087,251.6816 120.4731,251.9805 L120.4731,254.7031 Q119.8423,254.1221 119.2488,253.8523 Q118.6553,253.5825 118.0244,253.5825 Q116.6797,253.5825 115.9949,254.6492 Q115.3101,255.7158 115.3101,257.8159 Q115.3101,259.9077 115.9949,260.9744 Q116.6797,262.041 118.0244,262.041 Q118.6553,262.041 119.2488,261.7712 Q119.8423,261.5015 120.4731,260.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="91" x="131.5" y="262.5352">sqlSessionProxy</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="103.5" x2="224.5" y1="274" y2="274"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="103.5" x2="224.5" y1="282" y2="282"></line><!--class SqlSessionFactory--><rect fill="#FEFECE" filter="url(#f1vwfwq1xir8hg)" height="48" id="SqlSessionFactory" style="stroke: #A80036; stroke-width: 1.5;" width="132" x="261" y="242"></rect><ellipse cx="276" cy="258" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M278.9731,263.6431 Q278.3921,263.9419 277.7529,264.0913 Q277.1138,264.2407 276.4082,264.2407 Q273.9014,264.2407 272.5815,262.5889 Q271.2617,260.937 271.2617,257.8159 Q271.2617,254.6865 272.5815,253.0347 Q273.9014,251.3828 276.4082,251.3828 Q277.1138,251.3828 277.7612,251.5322 Q278.4087,251.6816 278.9731,251.9805 L278.9731,254.7031 Q278.3423,254.1221 277.7488,253.8523 Q277.1553,253.5825 276.5244,253.5825 Q275.1797,253.5825 274.4949,254.6492 Q273.8101,255.7158 273.8101,257.8159 Q273.8101,259.9077 274.4949,260.9744 Q275.1797,262.041 276.5244,262.041 Q277.1553,262.041 277.7488,261.7712 Q278.3423,261.5015 278.9731,260.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="100" x="290" y="262.5352">SqlSessionFactory</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="262" x2="392" y1="274" y2="274"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="262" x2="392" y1="282" y2="282"></line><!--link SqlSession to DefaultSqlSession--><path d="M83.9072,76.1055 C81.3525,90.0291 78.6252,104.8927 76.4262,116.877 " fill="none" id="SqlSession-DefaultSqlSession" style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 7.0,7.0;"></path><polygon fill="none" points="77.0667,74.5989,87.5614,56.1906,90.8369,77.1256,77.0667,74.5989" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link SqlSession to SqlSessionTemplate--><path d="M142.7137,67.8948 C165.4234,83.9685 191.5997,102.4959 211.9179,116.877 " fill="none" id="SqlSession-SqlSessionTemplate" style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 7.0,7.0;"></path><polygon fill="none" points="138.4582,73.4588,126.1776,56.1906,146.5463,62.0315,138.4582,73.4588" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link SqlSessionTemplate to sqlSessionProxy--><path d="M225.9744,165.1636 C218.7409,174.2925 210.7102,184.9138 204,195 C195.0662,208.4284 186.1569,223.9767 179.022,237.0939 " fill="none" id="SqlSessionTemplate-sqlSessionProxy" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="#A80036" points="176.4768,241.8081,184.2723,235.789,178.8522,237.4084,177.2328,231.9883,176.4768,241.8081" style="stroke: #A80036; stroke-width: 1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="67" x="205" y="208.5684">SqlSession</text><!--link SqlSessionTemplate to SqlSessionFactory--><path d="M261.6176,165.1013 C275.0644,185.8525 294.4861,215.8243 308.6673,237.7088 " fill="none" id="SqlSessionTemplate-SqlSessionFactory" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="#A80036" points="311.4303,241.9727,309.893,232.2446,308.7113,237.7766,303.1793,236.5949,311.4303,241.9727" style="stroke: #A80036; stroke-width: 1.0;"></polygon></g></svg>

Executor

<!--
```plantuml
interface Executor

Executor <|.. BaseExecutor
BaseExecutor <|-- BatchExecutor
BaseExecutor <|-- SimpleExecutor
BaseExecutor <|-- ReuseExecutor
```
-->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="283px" preserveAspectRatio="none" style="width:438px;height:283px;" version="1.1" viewBox="0 0 438 283" width="438px" zoomAndPan="magnify"><defs><filter height="300%" id="fdfz5u8kgoels" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><!--class Executor--><rect fill="#FEFECE" filter="url(#fdfz5u8kgoels)" height="48" id="Executor" style="stroke: #A80036; stroke-width: 1.5;" width="82" x="173.5" y="8"></rect><ellipse cx="188.5" cy="24" fill="#B4A7E5" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M184.4277,19.7651 L184.4277,17.6069 L191.8071,17.6069 L191.8071,19.7651 L189.3418,19.7651 L189.3418,27.8418 L191.8071,27.8418 L191.8071,30 L184.4277,30 L184.4277,27.8418 L186.8931,27.8418 L186.8931,19.7651 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" font-style="italic" lengthAdjust="spacingAndGlyphs" textLength="50" x="202.5" y="28.5352">Executor</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="174.5" x2="254.5" y1="40" y2="40"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="174.5" x2="254.5" y1="48" y2="48"></line><!--class BaseExecutor--><rect fill="#FEFECE" filter="url(#fdfz5u8kgoels)" height="48" id="BaseExecutor" style="stroke: #A80036; stroke-width: 1.5;" width="109" x="160" y="116"></rect><ellipse cx="175" cy="132" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M177.9731,137.6431 Q177.3921,137.9419 176.7529,138.0913 Q176.1138,138.2407 175.4082,138.2407 Q172.9014,138.2407 171.5815,136.5889 Q170.2617,134.937 170.2617,131.8159 Q170.2617,128.6865 171.5815,127.0347 Q172.9014,125.3828 175.4082,125.3828 Q176.1138,125.3828 176.7612,125.5322 Q177.4087,125.6816 177.9731,125.9805 L177.9731,128.7031 Q177.3423,128.1221 176.7488,127.8523 Q176.1553,127.5825 175.5244,127.5825 Q174.1797,127.5825 173.4949,128.6492 Q172.8101,129.7158 172.8101,131.8159 Q172.8101,133.9077 173.4949,134.9744 Q174.1797,136.041 175.5244,136.041 Q176.1553,136.041 176.7488,135.7712 Q177.3423,135.5015 177.9731,134.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="77" x="189" y="136.5352">BaseExecutor</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="161" x2="268" y1="148" y2="148"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="161" x2="268" y1="156" y2="156"></line><!--class BatchExecutor--><rect fill="#FEFECE" filter="url(#fdfz5u8kgoels)" height="48" id="BatchExecutor" style="stroke: #A80036; stroke-width: 1.5;" width="113" x="6" y="224"></rect><ellipse cx="21" cy="240" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M23.9731,245.6431 Q23.3921,245.9419 22.7529,246.0913 Q22.1138,246.2407 21.4082,246.2407 Q18.9014,246.2407 17.5815,244.5889 Q16.2617,242.937 16.2617,239.8159 Q16.2617,236.6865 17.5815,235.0347 Q18.9014,233.3828 21.4082,233.3828 Q22.1138,233.3828 22.7612,233.5322 Q23.4087,233.6816 23.9731,233.9805 L23.9731,236.7031 Q23.3423,236.1221 22.7488,235.8523 Q22.1553,235.5825 21.5244,235.5825 Q20.1797,235.5825 19.4949,236.6492 Q18.8101,237.7158 18.8101,239.8159 Q18.8101,241.9077 19.4949,242.9744 Q20.1797,244.041 21.5244,244.041 Q22.1553,244.041 22.7488,243.7712 Q23.3423,243.5015 23.9731,242.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="81" x="35" y="244.5352">BatchExecutor</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="7" x2="118" y1="256" y2="256"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="7" x2="118" y1="264" y2="264"></line><!--class SimpleExecutor--><rect fill="#FEFECE" filter="url(#fdfz5u8kgoels)" height="48" id="SimpleExecutor" style="stroke: #A80036; stroke-width: 1.5;" width="120" x="154.5" y="224"></rect><ellipse cx="169.5" cy="240" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M172.4731,245.6431 Q171.8921,245.9419 171.2529,246.0913 Q170.6138,246.2407 169.9082,246.2407 Q167.4014,246.2407 166.0815,244.5889 Q164.7617,242.937 164.7617,239.8159 Q164.7617,236.6865 166.0815,235.0347 Q167.4014,233.3828 169.9082,233.3828 Q170.6138,233.3828 171.2612,233.5322 Q171.9087,233.6816 172.4731,233.9805 L172.4731,236.7031 Q171.8423,236.1221 171.2488,235.8523 Q170.6553,235.5825 170.0244,235.5825 Q168.6797,235.5825 167.9949,236.6492 Q167.3101,237.7158 167.3101,239.8159 Q167.3101,241.9077 167.9949,242.9744 Q168.6797,244.041 170.0244,244.041 Q170.6553,244.041 171.2488,243.7712 Q171.8423,243.5015 172.4731,242.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="88" x="183.5" y="244.5352">SimpleExecutor</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="155.5" x2="273.5" y1="256" y2="256"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="155.5" x2="273.5" y1="264" y2="264"></line><!--class ReuseExecutor--><rect fill="#FEFECE" filter="url(#fdfz5u8kgoels)" height="48" id="ReuseExecutor" style="stroke: #A80036; stroke-width: 1.5;" width="117" x="310" y="224"></rect><ellipse cx="325" cy="240" fill="#ADD1B2" rx="11" ry="11" style="stroke: #A80036; stroke-width: 1.0;"></ellipse><path d="M327.9731,245.6431 Q327.3921,245.9419 326.7529,246.0913 Q326.1138,246.2407 325.4082,246.2407 Q322.9014,246.2407 321.5815,244.5889 Q320.2617,242.937 320.2617,239.8159 Q320.2617,236.6865 321.5815,235.0347 Q322.9014,233.3828 325.4082,233.3828 Q326.1138,233.3828 326.7612,233.5322 Q327.4087,233.6816 327.9731,233.9805 L327.9731,236.7031 Q327.3423,236.1221 326.7488,235.8523 Q326.1553,235.5825 325.5244,235.5825 Q324.1797,235.5825 323.4949,236.6492 Q322.8101,237.7158 322.8101,239.8159 Q322.8101,241.9077 323.4949,242.9744 Q324.1797,244.041 325.5244,244.041 Q326.1553,244.041 326.7488,243.7712 Q327.3423,243.5015 327.9731,242.9204 Z "></path><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacingAndGlyphs" textLength="85" x="339" y="244.5352">ReuseExecutor</text><line style="stroke: #A80036; stroke-width: 1.5;" x1="311" x2="426" y1="256" y2="256"></line><line style="stroke: #A80036; stroke-width: 1.5;" x1="311" x2="426" y1="264" y2="264"></line><!--link Executor to BaseExecutor--><path d="M214.5,76.2911 C214.5,89.8171 214.5,104.1835 214.5,115.8436 " fill="none" id="Executor-BaseExecutor" style="stroke: #A80036; stroke-width: 1.0; stroke-dasharray: 7.0,7.0;"></path><polygon fill="none" points="207.5001,76.2373,214.5,56.2373,221.5001,76.2373,207.5001,76.2373" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link BaseExecutor to BatchExecutor--><path d="M163.8632,175.9788 C141.7178,191.7137 116.3272,209.7544 96.4979,223.8436 " fill="none" id="BaseExecutor-BatchExecutor" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="160.0301,170.1152,180.3882,164.2373,168.1391,181.5278,160.0301,170.1152" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link BaseExecutor to SimpleExecutor--><path d="M214.5,184.2911 C214.5,197.8171 214.5,212.1835 214.5,223.8436 " fill="none" id="BaseExecutor-SimpleExecutor" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="207.5001,184.2373,214.5,164.2373,221.5001,184.2373,207.5001,184.2373" style="stroke: #A80036; stroke-width: 1.0;"></polygon><!--link BaseExecutor to ReuseExecutor--><path d="M265.4717,175.7464 C287.9863,191.5358 313.8656,209.6849 334.0547,223.8436 " fill="none" id="BaseExecutor-ReuseExecutor" style="stroke: #A80036; stroke-width: 1.0;"></path><polygon fill="none" points="261.416,181.452,249.0606,164.2373,269.4545,169.9897,261.416,181.452" style="stroke: #A80036; stroke-width: 1.0;"></polygon></g></svg>

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

protected Connection getConnection(Log statementLog) throws SQLException {
  Connection connection = transaction.getConnection();
  if (statementLog.isDebugEnabled()) {
    return ConnectionLogger.newInstance(connection, statementLog, queryStack);
  } else {
    return connection;
  }
}
```

事务管理器：

* ManagedTransaction
* JdbcTransaction

##### 1. JdbcTransaction

```java
public class JdbcTransaction implements Transaction {

  public JdbcTransaction(DataSource ds, TransactionIsolationLevel desiredLevel, boolean desiredAutoCommit) {
    dataSource = ds;
    level = desiredLevel;
    autoCommmit = desiredAutoCommit;
  }

  public JdbcTransaction(Connection connection) {
    this.connection = connection;
  }

  @Override
  public void commit() throws SQLException {
    if (connection != null && !connection.getAutoCommit()) {
      if (log.isDebugEnabled()) {
        log.debug("Committing JDBC Connection [" + connection + "]");
      }
      connection.commit();
    }
  }
  ...

  @Override
  public void close() throws SQLException {
    if (connection != null) {
      resetAutoCommit();
      if (log.isDebugEnabled()) {
        log.debug("Closing JDBC Connection [" + connection + "]");
      }
      connection.close();
    }
  }

  @Override
  public Connection getConnection() throws SQLException {
    if (connection == null) {
      openConnection();
    }
    return connection;
  }

  protected void openConnection() throws SQLException {
    if (log.isDebugEnabled()) {
      log.debug("Opening JDBC Connection");
    }
    connection = dataSource.getConnection();
    if (level != null) {
      connection.setTransactionIsolation(level.getLevel());
    }
    setDesiredAutoCommit(autoCommmit);
  }
}
```

##### 2. ManagedTransaction

#### 2.3.2 Spring管理


```plantuml

TransactionFactory <|.. SpringManagedTransactionFactory
SpringManagedTransactionFactory ..> SpringManagedTransaction
SpringManagedTransaction --|> Transaction

class SpringManagedTransaction {
  - openConnection()
}
note left: 调用Spring的DataSourceUtils\n.getConnection()\n获取连接
```

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

### 2.4 插件

```java
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

## 3. 缓存