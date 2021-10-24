# ClickHouse

[ClickHouse](https://clickhouse.tech/)
https://clickhouse.tech/docs/en/

[install](https://clickhouse.tech/docs/en/getting-started/install/)
[docker](https://hub.docker.com/r/yandex/clickhouse-server/)

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

JdbcTemplate 查询对象时使用 `DataClassRowMapper` 处理字段和属性映射, `org.springframework.jdbc.core.BeanPropertyRowMapper#initialize` 缓存小写转化后的属性和下划线分割处理后的属性，这样可以同时支持多种命名风格

```java
this.mappedFields.put(lowerCaseName(pd.getName()), pd);
String underscoredName = underscoreName(pd.getName());
if (!lowerCaseName(pd.getName()).equals(underscoredName)) {
	this.mappedFields.put(underscoredName, pd);
}
```


querydsl

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

资料:

* [ClickHouse内核分析-MergeTree的Merge和Mutation机制](https://developer.aliyun.com/article/762090?spm=a2c6h.12873581.0.0.29cc802f1GeMHc&groupCode=clickhouse)