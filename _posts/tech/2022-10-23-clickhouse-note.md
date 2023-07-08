---
layout: post
category : Tech
title : 基于 ClickHouse 的数据统计平台设计简述
tags : [spring, java, clickhouse, querydsl]
excerpt: 近年来大数据统计需求越来越多，基于此业务的数据库如雨后春笋般闪现，ClickHouse 就是此方面的重要明星，那么该如何设计一套符合业务的大数据统计系统？
---
{% include JB/setup %}

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 为何选择 ClickHouse](#1-为何选择-clickhouse)
- [2. 平台设计](#2-平台设计)
  - [2.1 技术栈](#21-技术栈)
  - [2.2 业务实现](#22-业务实现)
    - [2.2.1 入库](#221-入库)
    - [2.2.2 查询](#222-查询)
- [3. 参考资料](#3-参考资料)
- [4. 常见问题](#4-常见问题)
  - [4.1 ClickHouse 安装配置](#41-clickhouse-安装配置)
  - [4.2 数据库的数据合并操作比插入慢](#42-数据库的数据合并操作比插入慢)
  - [4.3 部分数据遗漏](#43-部分数据遗漏)
  - [4.4 常用配置](#44-常用配置)
  - [4.5 数据初始化脚本](#45-数据初始化脚本)

<!-- /code_chunk_output -->


近年来大数据统计需求越来越多，基于此业务的数据库如雨后春笋般闪现，ClickHouse 就是此方面的重要明星，那么该如何设计一套符合业务的大数据统计系统？

本文以常见的站点用户访问统计系统为例，下文统一简写为统计系统。原始数据存储到 `MongoDB` 中，统计分析数据库使用 `ClickHouse`，数据流转中间件为 `Kafka`。

## 1. 为何选择 ClickHouse

统计系统需求:

* 针对大量字段的大表进行统计
* 部分字段类型为数组，因此最好支持数组，这样不需要拆分原有数据
* 针对时间等字段进行排序分组等，因此需要支持 `groupBy`, `orderBy` 等聚合操作
* 需要统计结果具有实时性
* 不需要严格的事务支持
* 很少需要更新操作
* 易于开发

针对以上需求，考察当前流行的 OLAP 系统后发现 ClickHouse 完美契合当前需求。ClickHouse 是一个用于联机分析(OLAP)的列式数据库管理系统(DBMS)。它支持高速读取和写入大数据量，并且可以通过水平扩展轻松地扩展到多个节点上。ClickHouse 适用于各种类型的数据分析，包括实时和流数据分析，数据仓库等。OLAP场景的关键特征:

* 绝大多数是读请求
* 数据以相当大的批次插入
* 对于读取，从数据库中提取相当多的行，但只提取列的一小部分。
* 宽表，即每个表包含着大量的列
* 查询相对较少(通常每台服务器每秒查询数百次或更少)
* 对于简单查询，允许适当延迟
* 列中的数据长度相对较小
* 处理单个查询时需要高吞吐量(每台服务器每秒可达数十亿行)
* 事务不是必须的
* 对数据一致性要求低
* 每个查询有一个大表。除了他以外，其他的都很小。
* 查询结果明显小于源数据。换句话说，数据经过过滤或聚合，因此单个普通服务器也可以支持

很容易可以看出，OLAP 场景与其他通常业务场景(例如, OLTP 或 K/V )有很大的不同， 因此想要使用 OLTP 或 Key-Value 数据库去高效的处理分析查询场景，并不是非常完美的适用方案。例如，使用 OLAP 数据库去处理分析请求通常要优于使用 MongoDB 或 Redis 去处理分析请求。ClickHouse 的各项特点更为符合统计系统统计需求，因此采用 ClickHouse 数据库作为承载统计需求的底层数据库。

用户访问统计系统的业务需要实时或全量的方式把数导入到 ClickHouse 中。

## 2. 平台设计

### 2.1 技术栈

由于官方只提供了基本的 JDBC 驱动，目前还没有对应的 Spring Data 模块，为操作方便使用 **Spring JdbcTemplate** 获取 `Connection/Statement`，减少直接操作 `JDBC` 。

统计指标查询使用 **Querydsl**，虽然 ClickHouse 提供了 JDBC 驱动，但是由于它独有的一些数据结构导致不兼容 ORM 框架，无论是 JPA 还是 Myatis，要想实现大量的查询需要编写大量的 SQL 语句，这样导致开发困难，而且bug率很增加，为了避免这些情况，决定使用 [Querydsl](https://querydsl.com/)。

Querydsl 是一个通用的查询框架，专注于通过 Java Api 构建类型安全的SQL查询。它可以通过一组通用的查询 Api 为用户构建出适合不同类型 ORM 框架或者是SQL 的查询语句，也就是说 Querydsl 是基于各种 ORM 框架以及 SQL 之上的一个通用的查询框架。借助 Querydsl 可以在任何支持的 ORM 框架或者 SQL 平台上以一种通用的 API 方式来构建查询。如果通过 Java Api 方式开发，可把大量查询操作封装，统一注入条件和参数等，这样即提高了开发效率，降低了 bug 率，而且提供了更好的封装性，不再受直接写 SQL 语句开发困难影响。

在本例中通过扩展 `querydsl-sql`，针对数组，时间等数据类型进行自定义，支持 `arrayJoin`、`has` 等函数，最大程度上提供开箱即用的 ClickHouse Java Api 开发。

查询用户访问页面 TOP 10 示例:

```java
query
   .select(QUserVisit.userVisit.path.countDistinct().as(COUNT))
   .orderBy(COUNT_EXPRESSION.desc())
   .limit(10)
   .fetch();
```

### 2.2 业务实现

#### 2.2.1 入库

统计系统数据处理流程：当收集到统计数据后发送数据到数据清洗消息队列（Kafka），数据经过清洗，标记等一系列流程后发送到数据归档消息队列，由入库程序负责把消息写入数据库（ClickHouse）。

分析上述流程，统计数据可以分为两类：

* Kafka 实时数据
* MongoDB 历史数据

这两种数据虽然来源不同，但是数据结构基本一致，都可以抽象为相同的数据结构，因此可以采用相同入库方式。数据层通过抽象，封装写操作保证接口一致，内部实现批量写入，这样不仅易于开发而且提高写入效率。

<?xml version="1.0" encoding="UTF-8" standalone="no"?><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentScriptType="application/ecmascript" contentStyleType="text/css" height="180px" preserveAspectRatio="none" style="width:313px;height:180px;" version="1.1" viewBox="0 0 313 180" width="313px" zoomAndPan="magnify"><defs><filter height="300%" id="f1gnxo2tin6672" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"/><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"/><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"/><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"/></filter></defs><g><text fill="#000000" font-family="sans-serif" font-size="18" lengthAdjust="spacing" textLength="72" x="118" y="28.708">入库流程</text><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="35" x2="35" y1="75.25" y2="137.25"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="105" x2="105" y1="75.25" y2="137.25"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="189" x2="189" y1="75.25" y2="137.25"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="273" x2="273" y1="75.25" y2="137.25"/><rect fill="#FEFECE" filter="url(#f1gnxo2tin6672)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="5" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="12" y="59.9482">数据源</text><rect fill="#FEFECE" filter="url(#f1gnxo2tin6672)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="5" y="136.25"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="12" y="156.2451">数据源</text><rect fill="#FEFECE" filter="url(#f1gnxo2tin6672)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="75" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="82" y="59.9482">处理层</text><rect fill="#FEFECE" filter="url(#f1gnxo2tin6672)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="75" y="136.25"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="82" y="156.2451">处理层</text><rect fill="#FEFECE" filter="url(#f1gnxo2tin6672)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="84" x="145" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="70" x="152" y="59.9482">数据写入层</text><rect fill="#FEFECE" filter="url(#f1gnxo2tin6672)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="84" x="145" y="136.25"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="70" x="152" y="156.2451">数据写入层</text><rect fill="#FEFECE" filter="url(#f1gnxo2tin6672)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="243" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="250" y="59.9482">数据库</text><rect fill="#FEFECE" filter="url(#f1gnxo2tin6672)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="243" y="136.25"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="250" y="156.2451">数据库</text><polygon fill="#A80036" points="93,87.25,103,91.25,93,95.25,97,91.25" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="35" x2="99" y1="91.25" y2="91.25"/><polygon fill="#A80036" points="177,101.25,187,105.25,177,109.25,181,105.25" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="105" x2="183" y1="105.25" y2="105.25"/><polygon fill="#A80036" points="261,115.25,271,119.25,261,123.25,265,119.25" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="189" x2="267" y1="119.25" y2="119.25"/></g></svg>

**实时数据**: 实时数据来源为 Kafka 消息队列，当客户端获取到消息后发送给 `DataWriter`, `DataWriter` 负责把消息数据转换为 JSON 格式并通过 `ClickHouseFormat.JSONEachRow` 方式入库。插入数据时需要判断数据是否存在数据库中，如果存在则把该数据加入删除队列（*如果使用 ReplacingMergeTree 者不需要删除操作*）。

提高入库效率措施:

* 生产者批量发送和启用消息压缩以增加单批次获取的消息量
* 采用布隆过滤器来实现判断数据是否在数据库中，这样不仅内存消耗小，而且效率高
* 后台任务定时从删除队列获取数据并合并为单条 SQL 语句以执行删除操作


**数据更新**: 由于 ClickHouse 并不支持常规意义上的数据更新操作，而后通过后台线程来刷新合并旧数据，因此为了保证查询准确性，插入数据前先判断是否存在旧数据，如存在则删除旧数据然后插入新数据（*不推荐删除操作，可使用 ReplacingMergeTree 等表引擎实现*）。

插入数据前使用用户唯一标识和 "dataTime 小于当前时间(时间精确度至少为微秒)" 作为删除数据条件，插入该条件到删除任务队列。

注意: Java 提供的 `SimpleDateFormat` 日期格式化器只能序列化到毫秒，`DateTimeFormatter` 可以精确到纳秒。推荐使用 `LocalDateTime`， JDK 版本高于8时精确到微秒。


**全量数据**: 导入全量数据时，清空数据库和布隆过滤器数据，清空删除任务队列，接下来的流程与实时保持一致。如果第一次做全量导入则全量数据的布隆过滤器创建成功，要是不做则说明是全新的系统则布隆过滤器从头开始，以上两种情况都下的布隆过滤器都存储完整数据，确保极低的数据重复率。

从数据库导入数据流程:

1. 重置数据库
2. 导入数据，具体实现可通过 `Mongotemplat#stream` 方式，合并多条数据为一个批次，其它流程和 Kafka 一致，注意记录相关日志信息以及导入进度

全量导入时，也可采用临时表方式，全量数据导入到临时表，实时数据写入切换到临时表，旧表不变，等导入完成后，旧表改名为临时表，新表改为原表名，删除临时表，注意重命名时不能做修改数据等操作。

写数据时以 JSON 格式写入，自定义 `NestedList` 数据类型以适配 `Nested` 类型，通过 `NestedListSerializer` 把 `NestedList` 数据转换为 ClickHouse 可识别的 JSON 结构，通过以下方式把序列化器注入到 Jackson 中。`

设计表时的注意点:

1. 字段命名调整为小驼峰风格，便于转为 JSON 批量写入
2. 每个表添加一个 `dataTime` 字段，表示数据插入时间，精确到微秒， 数据类型为 `datetime64`
3. 数组结构使用 `Array` 数据类型
4. 数组內部元素为对象时使用 `Nested` 类型(特殊的数组)


数据入库使用`DataWriter`, 该接口定义数据写入规范，`DataRecord` 封装数据和时间戳，批次号等数据，当抛出异常时，交由 `ErrorReporter` 处理，默认提供日志打印功能，用户可自定义该操作，无论是重试还是发送到其它消息队列。

<?xml version="1.0" encoding="UTF-8" standalone="no"?><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentScriptType="application/ecmascript" contentStyleType="text/css" height="660px" preserveAspectRatio="none" style="width:948px;height:660px;" version="1.1" viewBox="0 0 948 660" width="948px" zoomAndPan="magnify"><defs><filter height="300%" id="frxuyfo3kfp6h" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"/><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"/><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"/><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"/></filter></defs><g><text fill="#000000" font-family="sans-serif" font-size="18" lengthAdjust="spacing" textLength="108" x="417.75" y="28.708">数据写入流程</text><rect fill="#ADD8E6" height="613.8516" style="stroke:#A80036;stroke-width:1.0;" width="247" x="1" y="40.9531"/><text fill="#000000" font-family="sans-serif" font-size="13" font-weight="bold" lengthAdjust="spacing" textLength="52" x="96" y="53.02">应用服务</text><rect fill="#8FBC8F" height="613.8516" style="stroke:#A80036;stroke-width:1.0;" width="493.5" x="250" y="40.9531"/><text fill="#000000" font-family="sans-serif" font-size="13" font-weight="bold" lengthAdjust="spacing" textLength="78" x="455.25" y="53.02">数据引擎服务</text><rect fill="#DDDDDD" height="613.8516" style="stroke:#A80036;stroke-width:1.0;" width="197" x="745.5" y="40.9531"/><text fill="#000000" font-family="sans-serif" font-size="13" font-weight="bold" lengthAdjust="spacing" textLength="78" x="805" y="53.02">全量导入任务</text><rect fill="#FFFFFF" filter="url(#frxuyfo3kfp6h)" height="384.5938" style="stroke:#A80036;stroke-width:1.0;" width="10" x="482.5" y="184.7813"/><rect fill="#FFFFFF" filter="url(#frxuyfo3kfp6h)" height="98.5313" style="stroke:#A80036;stroke-width:1.0;" width="10" x="781.5" y="441.7109"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="70" x2="70" y1="95.3828" y2="616.5078"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="194" x2="194" y1="95.3828" y2="616.5078"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="319" x2="319" y1="95.3828" y2="616.5078"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="487.5" x2="487.5" y1="95.3828" y2="616.5078"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="695.5" x2="695.5" y1="95.3828" y2="616.5078"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="786.5" x2="786.5" y1="95.3828" y2="616.5078"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="901.5" x2="901.5" y1="95.3828" y2="616.5078"/><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="126" x="5" y="60.0859"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="112" x="12" y="80.0811">应用统计数据收集</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="126" x="5" y="615.5078"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="112" x="12" y="635.5029">应用统计数据收集</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="95" x="145" y="60.0859"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="81" x="152" y="80.0811">KafkaServer</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="95" x="145" y="615.5078"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="81" x="152" y="635.5029">KafkaServer</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="127" x="254" y="60.0859"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="113" x="261" y="80.0811">MessageService</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="127" x="254" y="615.5078"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="113" x="261" y="635.5029">MessageService</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="86" x="442.5" y="60.0859"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="72" x="449.5" y="80.0811">DataWriter</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="86" x="442.5" y="615.5078"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="72" x="449.5" y="635.5029">DataWriter</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="84" x="651.5" y="60.0859"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="70" x="658.5" y="80.0811">数据库引擎</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="84" x="651.5" y="615.5078"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="70" x="658.5" y="635.5029">数据库引擎</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="70" x="749.5" y="60.0859"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="56" x="756.5" y="80.0811">读取数据</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="70" x="749.5" y="615.5078"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="56" x="756.5" y="635.5029">读取数据</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="70" x="864.5" y="60.0859"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="56" x="871.5" y="80.0811">开始任务</text><rect fill="#FEFECE" filter="url(#frxuyfo3kfp6h)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="70" x="864.5" y="615.5078"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="56" x="871.5" y="635.5029">开始任务</text><rect fill="#FFFFFF" filter="url(#frxuyfo3kfp6h)" height="384.5938" style="stroke:#A80036;stroke-width:1.0;" width="10" x="482.5" y="184.7813"/><rect fill="#FFFFFF" filter="url(#frxuyfo3kfp6h)" height="98.5313" style="stroke:#A80036;stroke-width:1.0;" width="10" x="781.5" y="441.7109"/><polygon fill="#A80036" points="182.5,122.5156,192.5,126.5156,182.5,130.5156,186.5,126.5156" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="70" x2="188.5" y1="126.5156" y2="126.5156"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="78" x="77" y="121.4497">发送统计数据</text><polygon fill="#A80036" points="307.5,151.6484,317.5,155.6484,307.5,159.6484,311.5,155.6484" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="194.5" x2="313.5" y1="155.6484" y2="155.6484"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="52" x="201.5" y="150.5825">接受消息</text><polygon fill="#A80036" points="470.5,180.7813,480.5,184.7813,470.5,188.7813,474.5,184.7813" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="319.5" x2="476.5" y1="184.7813" y2="184.7813"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="144" x="326.5" y="179.7153">转换消息为 DataRecord</text><line style="stroke:#A80036;stroke-width:1.0;" x1="492.5" x2="534.5" y1="213.9141" y2="213.9141"/><line style="stroke:#A80036;stroke-width:1.0;" x1="534.5" x2="534.5" y1="213.9141" y2="226.9141"/><line style="stroke:#A80036;stroke-width:1.0;" x1="493.5" x2="534.5" y1="226.9141" y2="226.9141"/><polygon fill="#A80036" points="503.5,222.9141,493.5,226.9141,503.5,230.9141,499.5,226.9141" style="stroke:#A80036;stroke-width:1.0;"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="78" x="499.5" y="208.8481">转换应用数据</text><line style="stroke:#A80036;stroke-width:1.0;" x1="492.5" x2="534.5" y1="256.0469" y2="256.0469"/><line style="stroke:#A80036;stroke-width:1.0;" x1="534.5" x2="534.5" y1="256.0469" y2="269.0469"/><line style="stroke:#A80036;stroke-width:1.0;" x1="493.5" x2="534.5" y1="269.0469" y2="269.0469"/><polygon fill="#A80036" points="503.5,265.0469,493.5,269.0469,503.5,273.0469,499.5,269.0469" style="stroke:#A80036;stroke-width:1.0;"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="184" x="499.5" y="250.981">根据 BloomFilter 判断是否存在</text><line style="stroke:#A80036;stroke-width:1.0;" x1="492.5" x2="534.5" y1="298.1797" y2="298.1797"/><line style="stroke:#A80036;stroke-width:1.0;" x1="534.5" x2="534.5" y1="298.1797" y2="311.1797"/><line style="stroke:#A80036;stroke-width:1.0;" x1="493.5" x2="534.5" y1="311.1797" y2="311.1797"/><polygon fill="#A80036" points="503.5,307.1797,493.5,311.1797,503.5,315.1797,499.5,311.1797" style="stroke:#A80036;stroke-width:1.0;"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="143" x="499.5" y="293.1138">已存在数据插入删除队列</text><polygon fill="#A80036" points="683.5,336.3125,693.5,340.3125,683.5,344.3125,687.5,340.3125" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="492.5" x2="689.5" y1="340.3125" y2="340.3125"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="169" x="499.5" y="335.2466">写入到 ClickHouse 等数据库</text><polygon fill="#A80036" points="503.5,365.4453,493.5,369.4453,503.5,373.4453,499.5,369.4453" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="497.5" x2="694.5" y1="369.4453" y2="369.4453"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="52" x="509.5" y="364.3794">返回结果</text><polygon fill="#A80036" points="330.5,394.5781,320.5,398.5781,330.5,402.5781,326.5,398.5781" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="324.5" x2="481.5" y1="398.5781" y2="398.5781"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="52" x="336.5" y="393.5122">处理结果</text><polygon fill="#A80036" points="205.5,423.7109,195.5,427.7109,205.5,431.7109,201.5,427.7109" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="199.5" x2="318.5" y1="427.7109" y2="427.7109"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="91" x="211.5" y="422.645">提交消息偏移量</text><polygon fill="#A80036" points="802.5,437.7109,792.5,441.7109,802.5,445.7109,798.5,441.7109" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="796.5" x2="900.5" y1="441.7109" y2="441.7109"/><line style="stroke:#A80036;stroke-width:1.0;" x1="791.5" x2="833.5" y1="484.4766" y2="484.4766"/><line style="stroke:#A80036;stroke-width:1.0;" x1="833.5" x2="833.5" y1="484.4766" y2="497.4766"/><line style="stroke:#A80036;stroke-width:1.0;" x1="792.5" x2="833.5" y1="497.4766" y2="497.4766"/><polygon fill="#A80036" points="802.5,493.4766,792.5,497.4766,802.5,501.4766,798.5,497.4766" style="stroke:#A80036;stroke-width:1.0;"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="91" x="798.5" y="479.4106">遍历数据库数据</text><path d="M629,454.7109 L629,509.7109 L772,509.7109 L772,464.7109 L762,454.7109 L629,454.7109 " fill="#FBFB77" filter="url(#frxuyfo3kfp6h)" style="stroke:#A80036;stroke-width:1.0;"/><path d="M762,454.7109 L762,464.7109 L772,464.7109 L762,454.7109 " fill="#FBFB77" style="stroke:#A80036;stroke-width:1.0;"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="111" x="635" y="471.7778">1. 重置数据库数据;</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="122" x="635" y="486.9106">2. 重置 BloomFilter;</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="68" x="635" y="502.0435">3. 遍历数据</text><polygon fill="#A80036" points="503.5,536.2422,493.5,540.2422,503.5,544.2422,499.5,540.2422" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="497.5" x2="785.5" y1="540.2422" y2="540.2422"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="144" x="509.5" y="535.1763">转换消息为 DataRecord</text><polygon fill="#A80036" points="774.5,565.375,784.5,569.375,774.5,573.375,778.5,569.375" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="487.5" x2="780.5" y1="569.375" y2="569.375"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="52" x="494.5" y="564.3091">返回结果</text><polygon fill="#A80036" points="889.5,594.5078,899.5,598.5078,889.5,602.5078,893.5,598.5078" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="786.5" x2="895.5" y1="598.5078" y2="598.5078"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="52" x="793.5" y="593.4419">任务结束</text></g></svg>


`DataWriter` 定义:

```java
public interface DataWriter<T> {

    void write(DataRecord<T> record);

    interface ErrorReporter {

        Future<Void> report(DataRecord<?> record, Throwable error);

    }
}
```
#### 2.2.2 查询

当前统计有大量的统计指标，为了简化开发难度，增强扩展性，需要抽象统计维度，定义统计维度接口，每个维度实现类只需实现查询。数据处理层负责汇总各个维度数据，注入查询参数等。调用方查询时只需传入维度键和参数即可，如果想要扩展统计维度，只需实现查询业务即可，不需关心业务之外的编码，这样极大的降低了开发难度，提高了开发效率。

<?xml version="1.0" encoding="UTF-8" standalone="no"?><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentScriptType="application/ecmascript" contentStyleType="text/css" height="335px" preserveAspectRatio="none" style="width:400px;height:335px;" version="1.1" viewBox="0 0 400 335" width="400px" zoomAndPan="magnify"><defs><filter height="300%" id="fk6oyrihscqeo" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"/><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"/><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"/><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"/></filter></defs><g><text fill="#000000" font-family="sans-serif" font-size="18" lengthAdjust="spacing" textLength="72" x="166.5" y="28.708">查询流程</text><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="35" x2="35" y1="75.25" y2="291.9141"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="111" x2="111" y1="75.25" y2="291.9141"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="209" x2="209" y1="75.25" y2="291.9141"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="307" x2="307" y1="75.25" y2="291.9141"/><rect fill="#FEFECE" filter="url(#fk6oyrihscqeo)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="5" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="12" y="59.9482">调用方</text><rect fill="#FEFECE" filter="url(#fk6oyrihscqeo)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="5" y="290.9141"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="12" y="310.9092">调用方</text><rect fill="#FEFECE" filter="url(#fk6oyrihscqeo)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="81" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="88" y="59.9482">处理层</text><rect fill="#FEFECE" filter="url(#fk6oyrihscqeo)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="81" y="290.9141"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="88" y="310.9092">处理层</text><rect fill="#FEFECE" filter="url(#fk6oyrihscqeo)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="112" x="151" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="98" x="158" y="59.9482">统计维度实现层</text><rect fill="#FEFECE" filter="url(#fk6oyrihscqeo)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="112" x="151" y="290.9141"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="98" x="158" y="310.9092">统计维度实现层</text><rect fill="#FEFECE" filter="url(#fk6oyrihscqeo)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="277" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="284" y="59.9482">数据库</text><rect fill="#FEFECE" filter="url(#fk6oyrihscqeo)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="56" x="277" y="290.9141"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="42" x="284" y="310.9092">数据库</text><polygon fill="#A80036" points="99,102.3828,109,106.3828,99,110.3828,103,106.3828" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="35" x2="105" y1="106.3828" y2="106.3828"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="52" x="42" y="101.3169">统计维度</text><polygon fill="#A80036" points="197,144.082,207,148.082,197,152.082,201,148.082" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="111" x2="203" y1="148.082" y2="148.082"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="65" x="118" y="143.0161">注入参数等</text><path d="M214,119.3828 L214,159.3828 L391,159.3828 L391,129.3828 L381,119.3828 L214,119.3828 " fill="#FBFB77" filter="url(#fk6oyrihscqeo)" style="stroke:#A80036;stroke-width:1.0;"/><path d="M381,119.3828 L381,129.3828 L391,129.3828 L381,119.3828 " fill="#FBFB77" style="stroke:#A80036;stroke-width:1.0;"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="156" x="220" y="136.4497">根据统计维度获取实现类，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="91" x="220" y="151.5825">执行查询和汇总</text><polygon fill="#A80036" points="295,170.6484,305,174.6484,295,178.6484,299,174.6484" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="209" x2="301" y1="174.6484" y2="174.6484"/><polygon fill="#A80036" points="220,184.6484,210,188.6484,220,192.6484,216,188.6484" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="214" x2="306" y1="188.6484" y2="188.6484"/><polygon fill="#A80036" points="122,198.6484,112,202.6484,122,206.6484,118,202.6484" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="116" x2="208" y1="202.6484" y2="202.6484"/><line style="stroke:#A80036;stroke-width:1.0;" x1="111" x2="153" y1="231.7813" y2="231.7813"/><line style="stroke:#A80036;stroke-width:1.0;" x1="153" x2="153" y1="231.7813" y2="244.7813"/><line style="stroke:#A80036;stroke-width:1.0;" x1="112" x2="153" y1="244.7813" y2="244.7813"/><polygon fill="#A80036" points="122,240.7813,112,244.7813,122,248.7813,118,244.7813" style="stroke:#A80036;stroke-width:1.0;"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="52" x="118" y="226.7153">汇总结果</text><polygon fill="#A80036" points="46,269.9141,36,273.9141,46,277.9141,42,273.9141" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="40" x2="110" y1="273.9141" y2="273.9141"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="52" x="52" y="268.8481">返回结果</text></g></svg>

`DataReader` 接口定义数据写入规范，`DataReaderKey` 定义统计维度键，`DataReaderKeys` 定义同时查询多维度时的键信息，`ItemDataReader` 接口定义具体统计维度实现，应用需实现该类来执行具体的查询操作。`QueryBuilder` 提供通用查询条件构建，可以把查询条件注入到查询中，统计维度实现类不需要自己实现该条件。

<?xml version="1.0" encoding="UTF-8" standalone="no"?><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" contentScriptType="application/ecmascript" contentStyleType="text/css" height="295px" preserveAspectRatio="none" style="width:483px;height:295px;" version="1.1" viewBox="0 0 483 295" width="483px" zoomAndPan="magnify"><defs><filter height="300%" id="f1efq73ic7gkfx" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"/><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"/><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"/><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"/></filter></defs><g><text fill="#000000" font-family="sans-serif" font-size="18" lengthAdjust="spacing" textLength="108" x="185.25" y="28.708">实时查询流程</text><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="42" x2="42" y1="75.25" y2="252.6484"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="126" x2="126" y1="75.25" y2="252.6484"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="254" x2="254" y1="75.25" y2="252.6484"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="365" x2="365" y1="75.25" y2="252.6484"/><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="453.5" x2="453.5" y1="75.25" y2="252.6484"/><rect fill="#FEFECE" filter="url(#f1efq73ic7gkfx)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="70" x="5" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="56" x="12" y="59.9482">前端服务</text><rect fill="#FEFECE" filter="url(#f1efq73ic7gkfx)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="70" x="5" y="251.6484"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="56" x="12" y="271.6436">前端服务</text><rect fill="#FEFECE" filter="url(#f1efq73ic7gkfx)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="70" x="89" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="56" x="96" y="59.9482">引擎接口</text><rect fill="#FEFECE" filter="url(#f1efq73ic7gkfx)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="70" x="89" y="251.6484"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="56" x="96" y="271.6436">引擎接口</text><rect fill="#FEFECE" filter="url(#f1efq73ic7gkfx)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="96" x="204" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="82" x="211" y="59.9482">DataReader</text><rect fill="#FEFECE" filter="url(#f1efq73ic7gkfx)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="96" x="204" y="251.6484"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="82" x="211" y="271.6436">DataReader</text><rect fill="#FEFECE" filter="url(#f1efq73ic7gkfx)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="98" x="314" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="84" x="321" y="59.9482">数据接查询层</text><rect fill="#FEFECE" filter="url(#f1efq73ic7gkfx)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="98" x="314" y="251.6484"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="84" x="321" y="271.6436">数据接查询层</text><rect fill="#FEFECE" filter="url(#f1efq73ic7gkfx)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="35" x="434.5" y="39.9531"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="21" x="441.5" y="59.9482">DB</text><rect fill="#FEFECE" filter="url(#f1efq73ic7gkfx)" height="30.2969" style="stroke:#A80036;stroke-width:1.5;" width="35" x="434.5" y="251.6484"/><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacing" textLength="21" x="441.5" y="271.6436">DB</text><polygon fill="#A80036" points="114,87.25,124,91.25,114,95.25,118,91.25" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="42" x2="120" y1="91.25" y2="91.25"/><polygon fill="#A80036" points="242,116.3828,252,120.3828,242,124.3828,246,120.3828" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="126" x2="248" y1="120.3828" y2="120.3828"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="104" x="133" y="115.3169">合并整合统计指标</text><polygon fill="#A80036" points="353,130.3828,363,134.3828,353,138.3828,357,134.3828" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="254" x2="359" y1="134.3828" y2="134.3828"/><polygon fill="#A80036" points="442,159.5156,452,163.5156,442,167.5156,446,163.5156" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="365" x2="448" y1="163.5156" y2="163.5156"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="65" x="372" y="158.4497">读取数据库</text><polygon fill="#A80036" points="376,173.5156,366,177.5156,376,181.5156,372,177.5156" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="370" x2="453" y1="177.5156" y2="177.5156"/><polygon fill="#A80036" points="265,187.5156,255,191.5156,265,195.5156,261,191.5156" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="259" x2="364" y1="191.5156" y2="191.5156"/><polygon fill="#A80036" points="137,216.6484,127,220.6484,137,224.6484,133,220.6484" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="131" x2="253" y1="220.6484" y2="220.6484"/><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="78" x="143" y="215.5825">汇总统计结果</text><polygon fill="#A80036" points="53,230.6484,43,234.6484,53,238.6484,49,234.6484" style="stroke:#A80036;stroke-width:1.0;"/><line style="stroke:#A80036;stroke-width:1.0;" x1="47" x2="125" y1="234.6484" y2="234.6484"/></g></svg>

相关接口定义:

```java
public interface DataReader {

    /**
     * 读根据指标取结果
     */
    <T> T read(DataReaderKey key, Object... condition);


    /**
     * 统计指标定义
     */
    interface DataReaderKey {

        String key();

    }


    /**
     * 具体的指标接口定义，内部使用
     */
    interface ItemDataReader<R> {

        Map<String, R> read(DataReaderKey key, Object... condition);

        DataReaderKey itemKey();

    }

    interface QueryBuilder<T> {

        /**
        * 构建查询条件
        */
        T build(T query, CountMode countMode);

    }
}

```

## 3. 参考资料

* [ClickHouse内核分析-MergeTree的Merge和Mutation机制](https://developer.aliyun.com/article/762090?spm=a2c6h.12873581.0.0.29cc802f1GeMHc&groupCode=clickhouse)
* [Githu Events 示例数据](https://github-sql.github.io/explorer/#download-the-dataset)

## 4. 常见问题

### 4.1 ClickHouse 安装配置

[ClickHouse](https://clickhouse.tech/) 安装参考文档:

* [安装文档](https://clickhouse.tech/docs/en/getting-started/install/)
* [DockerHub 镜像地址](https://hub.docker.com/r/yandex/clickhouse-server/)

安装相关命令:

**deb**

```shell
sudo apt-get install apt-transport-https ca-certificates dirmngr
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E0C56BD4

echo "deb https://repo.clickhouse.tech/deb/stable/ main/" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update

sudo apt-get install -y clickhouse-server clickhouse-client

sudo service clickhouse-server start
clickhouse-client
```

**rpm**

```shell
sudo yum install yum-utils
sudo rpm --import https://repo.clickhouse.tech/CLICKHOUSE-KEY.GPG
sudo yum-config-manager --add-repo https://repo.clickhouse.tech/rpm/stable/x86_64

sudo yum install clickhouse-server clickhouse-client

# Exception: Effective user of the process (root) does not match the owner of the data (clickhouse). Run under 'sudo -u clickhouse'
# 注意权限问题
sudo -u clickhouse clickhouse-server --config-file=/etc/clickhouse-server/config.xml
# 用户管理参考 https://clickhouse.com/docs/en/operations/settings/settings-users/
# 生成密码
PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'
```

**tar**
```shell
# 21.8.3.44
export LATEST_VERSION=`curl https://api.github.com/repos/ClickHouse/ClickHouse/tags 2>/dev/null | grep -Eo '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | head -n 1`
curl -O https://repo.clickhouse.tech/tgz/stable/clickhouse-common-static-$LATEST_VERSION.tgz
curl -O https://repo.clickhouse.tech/tgz/stable/clickhouse-common-static-dbg-$LATEST_VERSION.tgz
curl -O https://repo.clickhouse.tech/tgz/stable/clickhouse-server-$LATEST_VERSION.tgz
curl -O https://repo.clickhouse.tech/tgz/stable/clickhouse-client-$LATEST_VERSION.tgz

export LATEST_VERSION=21.8.3.44
tar -xzvf clickhouse-common-static-$LATEST_VERSION.tgz
sudo clickhouse-common-static-$LATEST_VERSION/install/doinst.sh

tar -xzvf clickhouse-common-static-dbg-$LATEST_VERSION.tgz
sudo clickhouse-common-static-dbg-$LATEST_VERSION/install/doinst.sh

tar -xzvf clickhouse-server-$LATEST_VERSION.tgz
sudo clickhouse-server-$LATEST_VERSION/install/doinst.sh
sudo /etc/init.d/clickhouse-server start

tar -xzvf clickhouse-client-$LATEST_VERSION.tgz
sudo clickhouse-client-$LATEST_VERSION/install/doinst.sh
```


### 4.2 数据库的数据合并操作比插入慢

参考 [DB::Exception: Too many parts (600). Merges are processing significantly slower than inserts](https://github.com/ClickHouse/ClickHouse/issues/3174), 控制单次插入数量为 10k-500k, 1-2s 插入一次，尽量合并小数据为大数据插入减少插入频率。

### 4.3 部分数据遗漏

`MongoTemplate#stream` 遍历的条件为插入时间，时间戳可能会重复，当排序字段的值大量相同且判断条件为大于时，可能导致有些数据被忽略，导致数据缺少，可通过增加一个自增批次号字段解决。

### 4.4 常用配置

编辑配置文件: `vim /etc/clickhouse-server/config.xml`(或 `/etc/clickhouse-server/config.d/listen.xml`)

允许非本机访问

```xml
<!-- Same for hosts without support for IPv6: -->
<listen_host>0.0.0.0</listen_host>
```

配置时区

```xml
<timezone>Asia/Shanghai</timezone>
```

### 4.5 数据初始化脚本

~~根据 Sql 文件初始化数据库(为密码安全 $2 可不写)~~，推荐使用 Flyway 等数据库自动升级工具。
```sh
#!/usr/bin/env bash
# 把单表的多行建表语句处理为一行并执行
sql_file=$1
passwd=$2
echo "start write $sql_file sql file"
cat $sql_file | xargs | tr '\n' ' ' | 
xargs | sed 's/;/;\n/g' | while read line; do
  if [ "$line" != '' ]; then
      clickhouse-client -q "$line" --password $passwd
  fi
done
```