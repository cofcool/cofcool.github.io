---
layout: post
category : Tech
title : 使用 Querydsl 快速开发基于 ClickHouse 的应用
tags : [spring, java, clickhouse, querydsl]
excerpt: 
---
{% include JB/setup %}

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. ClickHouse 安装配置](#1-clickhouse-安装配置)
- [2. ClickHouse 介绍](#2-clickhouse-介绍)
- [3. 应用开发](#3-应用开发)
  - [3.1 querydsl](#31-querydsl)
  - [3.2 应用示例](#32-应用示例)
- [4. 资料:](#4-资料)
- [5. 常见问题](#5-常见问题)
  - [5.1  Too many parts (300). Merges are processing significantly slower than inserts (version 21.8.3.44 (official build))](#51-too-many-parts-300-merges-are-processing-significantly-slower-than-inserts-version-218344-official-build)
  - [5.2 数据导入缺少数据问题](#52-数据导入缺少数据问题)

<!-- /code_chunk_output -->



## 1. ClickHouse 安装配置

[ClickHouse](https://clickhouse.tech/)

参考文档: [安装文档](https://clickhouse.tech/docs/en/getting-started/install/)，[docker 安装](https://hub.docker.com/r/yandex/clickhouse-server/)

安装相关命令:

```shell
# deb
sudo apt-get install apt-transport-https ca-certificates dirmngr
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E0C56BD4

echo "deb https://repo.clickhouse.tech/deb/stable/ main/" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update

sudo apt-get install -y clickhouse-server clickhouse-client

sudo service clickhouse-server start
clickhouse-client

# docker

# rpm

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

# tar
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

# client
sudo snap install dbeaver-ce

# multi-line to one line
# DB::Exception: Syntax error (Multi-statements are not allowed)
# so use the shell srcipt
cat script/init-table.sql | xargs | tr '\n' ' ' | xargs | sed 's/;/;\n/g' > init.sql
```

允许非本机访问, `vim /etc/clickhouse-server/config.xml`(`/etc/clickhouse-server/config.d/listen.xml`)
```xml
<!-- Same for hosts without support for IPv6: -->
<listen_host>0.0.0.0</listen_host>
```

配置时区
```xml
<timezone>Asia/Shanghai</timezone>
```

更改数据路径时注入目录权限

从sql文件初始化:
```sh
#!/usr/bin/env bash
sql_file=$1
passwd=$2
echo "start write $sql_file sql file"
cat $sql_file | xargs | tr '\n' ' ' | xargs | sed 's/;/;\n/g' | while read line; do
  if [ "$line" != '' ]; then
      clickhouse-client -q "$line" --password $passwd
  fi
done
```

example data

* [githu events](https://github-sql.github.io/explorer/#download-the-dataset)

## 2. ClickHouse 介绍

MergeTree:

* MergeTree
* ReplacingMergeTree
* SummingMergeTree
* AggregatingMergeTree
* CollapsingMergeTree
* VersionedCollapsingMergeTree
* GraphiteMergeTree


clickhouse projection 可预处理需要大量聚合场景


开发中直接使用 `JdbcTemplate` 来调用"JDBC 驱动"，提高开发效率

## 3. 应用开发

JdbcTemplate 查询对象时使用 `DataClassRowMapper` 处理字段和属性映射, `org.springframework.jdbc.core.BeanPropertyRowMapper#initialize` 缓存小写转化后的属性和下划线分割处理后的属性，这样可以同时支持多种命名风格

```java
this.mappedFields.put(lowerCaseName(pd.getName()), pd);
String underscoredName = underscoreName(pd.getName());
if (!lowerCaseName(pd.getName()).equals(underscoredName)) {
	this.mappedFields.put(underscoredName, pd);
}
```

### 3.1 querydsl

`com.querydsl.sql.AbstractSQLQuery#fetch` 调用 `com.querydsl.sql.AbstractSQLQuery#connection` 获取连接，执行完成后 `com.querydsl.sql.AbstractSQLQuery#endContext` 关闭连接，关闭通过 `com.querydsl.sql.SQLCloseListener#end` 完成

`com.querydsl.sql.Configuration#getListeners` 持有 listener

`SQLQueryFactory(SQLTemplates templates, Supplier<Connection> connection)` 方法创建的实例默认不会关闭连接，因此大量操作时会创建大量连接, 会导致消耗资源，占用内存以及超时等问题

```java
// com.querydsl.sql.SQLQueryFactory#query
// 注入 configuration
public SQLQuery<?> query() {
    return new SQLQuery<Void>(connection, configuration);
}

// com.querydsl.sql.AbstractSQLQuery#AbstractSQLQuery(java.util.function.Supplier<java.sql.Connection>, com.querydsl.sql.Configuration, com.querydsl.core.QueryMetadata)
// 注入 listeners 和 connProvider
// com.querydsl.sql.SQLCloseListener 关闭连接
public AbstractSQLQuery(Supplier<Connection> connProvider, Configuration configuration, QueryMetadata metadata) {
    super(new QueryMixin<Q>(metadata, false), configuration);
    this.connProvider = connProvider;
    this.listeners = new SQLListeners(configuration.getListeners());
    this.useLiterals = configuration.getUseLiterals();
}


// com.querydsl.sql.SQLQueryFactory#SQLQueryFactory(com.querydsl.sql.Configuration, javax.sql.DataSource, boolean)
// 直接注入 datasource, release 为 true 的话会配置 SQLCloseListener.DEFAULT，会在执行完 sql 后关闭连接
// 也可通过 configuration 手动添加
public SQLQueryFactory(Configuration configuration, DataSource dataSource, boolean release) {
    super(configuration, new DataSourceProvider(dataSource));
    if (release) {
        configuration.addListener(SQLCloseListener.DEFAULT);
    }
}


// 注入 connection
public SQLQueryFactory(SQLTemplates templates, Supplier<Connection> connection) {
    this(new Configuration(templates), connection);
}

```

### 3.2 应用示例

统计代码实现：
1. 定义datareader类，readData方法负责读取所有指标数据，参数定义为可变类型，readsingledata负责读取单个指标数据，参数多一个key
2. 定义itemdatareader子接口继承datareader，添加key和参数类型两个方法
2. 实现类内部每个统计指标实现一个itemdatatreader接口，并通过map持有所有指标类

itemdatareader不再继承datareader，另外增加一个result方法，负责把查找结果转换为map，另外datareader read相关方法只返回处理后的结果

添加where条件生成方法，所有查询都调用该方法

datareader.readdata可自定义指标组合即修改参数为map<指标，参数数组>，另外指标定义为枚举类型，增加key，参数类型属性

指标定义为接口，增加key，参数类型，组合指标等方法，itemdatareader继承此接口，read 方法增加指标参数，方便处理一个类包含多指标情况，移除result方法，read直接返回map

itemdatareader 不再继承指标接口，添加一个方法itemkey，返回值为指标接口类，这样方便处理多指标问题，在添加到readermap里时由于是字符串不知道它的组合指标是什么，因此不再使用字符串

注意⚠️，querydsl 工厂类注入连接的构造方法默认不会关闭连接，可改用带数据源参数的构造方法

datareader可考虑添加一个方法，同时读取多个指标，参数为map<key，args>，内部多线程执行，最后汇总结果。参数改为builder类，内部包含多个key：args结构，添加一个执行方法由它负责遍历以及组装返回结果，实现类为datareadkeys，已解决同一key不同参数情况，以及map作为参数构建实例复杂。注意key方法中判断是否存在相同key相同alias的值，如果则抛出异常

地区条件通过areaquerybuilder类完成，内部包含地区维度字段和地区信息，分组字段等，build（query）方法构建where条件，同一维度and拼接，不用维度or拼接，根据字段的metadata.name分组用or组合，分组内部判断读取那个地区信息and组合，实现的时候用括号（wrap）包住每一组条件，build时检测传入的字段是否是from指定的表的字段。reader里的getquery方法中添加地区条件，以便所有查询都包含地区条件。添加内部抽象类用于处理条件中的areabuilder条件，从可变参数中取出builder并传给子类，可变参数原样传给子类

指标类参数组合不需要定义areabuilder，默认含有该参数，位置不固定，参数检测时忽略检测areabuilder，注意参数下标

出现按省或地区分组的情况时，因为多个地区维度没办法分组，可以改为where查询条件，每个地区查一次接口，客户端可把结果缓存

开发者查询需要联表，当数据量大时数据库查询需要内存过多直接失败，因此在apkinfo表中添加developers字段，类型为数组

可把可变参数改为key-val数据结构，通过builder类实现，key为名字，val为类型，这样会导致传参数时还要定义名字，虽然可通过itemkey类定义但是这样更麻烦，还不如定义为可变参数，注释说明即可

querydsl 需要实现 hasAny（array）时需要自定义 serializer ，重写序列化常量方法即可，另外querydsl已提供基于collection的path，可以考虑重构一下charray

自定义spring jdbc rowmapper 时，click house的数组会实现java.sql.array，因此考虑添加一个basestatistics 添加一个方法array Meta 来把多个数组转换为一个nested list结构，这样的话只需参考spring 提供的数组转换实现，要是没有，自定义convert也不复杂，把该转换类注册到convert service即可，另外还需要重写dataclass row mapper方法以实现转换完成后把arraymeta合并分解为对应的nested list


应用统计表设计基本完成，写数据时以json格式写入，自定义nested数据类型，通过继承stdserilizer 重写序列化方法，拆解字段为数组形式，读取数据通过jdbctemplate，自定义rowmapper以实现nested以及多个字段变为内嵌对象的映射。经过测试发现这样做的话jsonunwrap失效了。另外想到一个办法，继承map，插入list数据时转换为对应的结构，即string：string[]。经过测试，jsonunwrap不对map生效，最后继承beanserilizer和unwrapserializer实现。如果要重写整个对象的序列化方式可通过继承beanserilizer实现，虽然此方法可行，但是需要处理全部字段过于复杂。

需要了解click house对数组的各种处理函数

字段命名调整为小驼峰风格，便于转为json批量写入，整体架构如前文所述，实现datawriter接口，当抛出特定异常时，交由错误报告程序处理。datarecord具体字段待定，目前是数据list和时间戳，批次号。特定异常改为任何异常，具体处理由实现类自己控制

所有表添加一个数据时间字段，值由数据库自动填充，表示数据插入时间，注意批次内部数据去重

数据库连接创建释放使用Spring jdbc 提供的工具类，查看源码发现没有必要使用工具类，datasourceutil是为事物服务，需要初始化事物绑定线程才会在该线程中重复利用该连接。spring data默认使用hikari作为连接池即不再需要其他手段了

Kafka消息批处理问题，目前一次获取的消息数量较少，看如何配置能提高数据量，从Spring和kafka client两处入手。经过测试发现还是不理想，无论自定义listener containerfactory还是调整Kafka参数，每批次拉取的数据数量还是较少，解决办法：生产者批量发送和启用消息压缩以及增加分区数量，手动调用Kafka客户端的poll以及手动提交（此方法过于麻烦，当现有性能不足时再考虑此方法，当然调整Spring kafka 参数也可以实现手动提交）

插入前根据md5和datatime删除旧数据，当然查询前去重也可以，需要注意的是删除数据的时间精确度至少为毫秒，因此数据类型为datetime64，精度为6，注意simple日期格式化器只能序列化到毫秒，time包下的序列化器可以精确到纳秒，测试发现local DateTime默认到豪秒，jdk版本高于8时精确到微秒。但是碰到新的问题有的旧数据被删除有的没，需要调试看问题所在，考虑采用异步队列后台删除模式。经过测试发现是JDBC驱动在转换时间为字符串时只精确到秒，忽略了毫秒等，因此删除时手动格式化时间为字符串不再通过JDBC驱动完成，经过测试生成的sql和预期的一致，另外决定使用后台定时删除操作合并多次删除为一次提高性能。不再使用时间作为判断条件，转为使用全局流水号作为判断，自定义一个生成器，保证所有数据都按插入顺序生成流水号,删除数据时batchno为主，md5为辅。经过思考，发现这么删除数据很麻烦而且容易错误的删除数据，是否可以采用后台任务进行 group by md5 where count > 1 计算的方式处理多余数据，数据回滚依然采用"batchNo"的方式。group by 的方式不可行，大量数据时不仅需要查询还要对比删除等操作更不可取，还是采用 batchNo + md5 的方式

插入数据时同时保存数据md5到一张很大的哈希表中，利用布隆过滤判断数据是否存在，应用程序启动时进行创建或者利用redis存储，这样可以避免无意义的删除操作，同时删除改为批量操作，条件为累计数据量和时间间隔取达到条件的其中之一即可。尝试批量删除时发现一个耗时问题，因为必须使用md5和datatime作为删除条件，导致每条数据生成一条SQL语句，多个表多条数据，从而使用sql语句过长，五十条数据会生成的250条SQL语句，并且执行耗时十秒左右，猜测是解析大量sql比较耗时，因此此方法不可取，还是采用单执行删除操作和布隆过滤结合使用。另外关闭程序时，接收到事件标记一下进入关闭模式，此时每接收到一次消息就删除一次，避免忽略部分数据(接收到关闭消息事件后还会继续收到几条kafka消息)。突然想到生成SQL语句不对，应该用or把数据拼成一条sql而不是一个条件一条sql，每批数据不要过大，目前50条，之后测试看看多少合适。其实没必要过早优化，因为批量删除会导致开发流程复杂，需要考虑各种边界条件，写完之后加个开关，可以切换到单次模式。为了保证删除条件之一日期正确，同一批次入库的数据持有相同dataTime，以及删除时条件是“小于dataTime”。目前删除采用接收到消息后判断是否执行删除操作，接下来改为定时任务自动执行删除操作，以及关机时执行删除任务保证队列为空。测试时发现数据部分删除了，刚开始以为是sql条件的问题，后来逐步分析代码发现是hashset引起的。因为deletedata的hash使用了md5字段的hash值，因此插入数据时因为key相同所以数据其实并没有真正插入进去，导致同一批次的相同md5的数据只删除了第一条，主要修改为map即可，key为md5，val为deletedata

可以尝试使用querydsl，基本没有问题，对于数组等数据类型自定义函数和sql template实现click house特有的函数等，常规查询目前没问题，但是多个数组联合条件怎么写还是个问题!querydsl添加自定义类型时注意创建方式，和其它默认类型保持一致，构造实例时需要把parent注入以及把自身添加到bean metadata 中

所有表添加day字段，只做展示，具体查询使用createtime做条件，以及createtime做分区字段，统计类增加父类，存储日期等字段

全量数据导入时，清空数据库和布隆过滤器数据，并缓存删除前的统计结果(似乎没必要)，以及清除待删除数据队列，接下来的流程与实时保持一致，如果第一次做全量导入则全量数据的布隆过滤器创建成功，要是不做则说明是全新的系统则布隆过滤器从头开始，以上两种情况都下的布隆过滤器都存储完整数据，确保极低的数据重复率。导入分为两种,一是直接从数据库读取，二是从kafka消费消息，这两种只有来源不一样，后续操作相同。从数据库导入数据时：1. 重置，2. 导入，实现可通过mongotemplate.stream 遍历数据，合并多条为一个批次，其它流程和Kafka一致，注意记录相关日志信息以及导入进度。另外全量导入时，可采用导入到tmp表，旧表不变，等导入完成后，旧表命名为tmp表，新表命名为原名，删除tmp表，注意重命名时不能做修改数据等操作

es入库通过rpc调用app-search实现，这样会增加无意义的调用链，为了避免es客户端依赖冲突，可clone es的源码通过更改包打自己的包来避免此问题

重置数据库加个开关，避免误删数据，双重验证

由于需要查询各地漏洞数量，联表apkinfo需要大量内存，暂时在漏洞表添加地区字段，以后有时间把漏洞和插件表合并到应用表，便于后续查询，通联url表添加地区字段，一个app下URL数量可能有数千个，是否需要合并到应用表？刚看了文档数组最大可支持一百万元素，待考虑。URL没必要合并，他有自己的归属，不强依赖应用，保持独立即可

数据引擎导入多种数据时，发生错误回滚数据实现参考 过滤链 或 事务，自定义事务管理器,增加管理器负责管理writer的各种状态，如是否开始执行，执行到哪个writer等，通过 threadlocal存储，注意相关数据的销毁，回滚失败时可以通过Kafka发送消息批次，也可通过事务完成方法来更新数据库状态

事务管理器回在回滚失败时通过同步器通知事务状态，在其中更新该批次数据状态

回滚操作时各个writer是否要记录本次事务插入的数据，如果这样，需要增加 comiit 方法来清除该数据，clickhouse 采用 batchNo 来删除本次事务中插入的数据，es 则需要判断该数据是否在数据库中，在则不做操作，不在则删除

DataWriters 中添加根据类型来调用 DataWriter 的方便方法

大量数据写入和删除时，click house会抛出文件合并过慢异常，因此增加是否启用 kafka 消息开关以及接口增加暂停时间配置

如果在任务中线程休眠会导致数据库连接超时，因此需要把每批次写入做成单独方法，本批次完成后在新的上下文中执行, 可通过定时任务实现，但是定时任务无法控制任务结束, 经过查看源码发现可以通过提交任务时返回的 future.cancel 方法取消任务

## 4. 资料:

* [ClickHouse内核分析-MergeTree的Merge和Mutation机制](https://developer.aliyun.com/article/762090?spm=a2c6h.12873581.0.0.29cc802f1GeMHc&groupCode=clickhouse)

## 5. 常见问题

### 5.1  Too many parts (300). Merges are processing significantly slower than inserts (version 21.8.3.44 (official build))

参考 [DB::Exception: Too many parts (600). Merges are processing significantly slower than inserts](https://github.com/ClickHouse/ClickHouse/issues/3174), 控制单次插入数量为 10k-500k, 1-2s 插入一次，尽量合并小数据为大数据插入减少插入频率，另外可以采用集群环境。

### 5.2 数据导入缺少数据问题

mongoTemplate.stream() 条件为 updateTime，，当 sort 字段的值大量相同且判断条件为大于时，可能导致有些数据被忽略，导致数据缺少。可通过增加一个自增批次号字段解决