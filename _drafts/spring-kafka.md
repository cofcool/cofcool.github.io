下载

```sh
https://www.apache.org/dyn/closer.cgi?path=/kafka/2.0.0/kafka_2.11-2.0.0.tgz

tar -xzf kafka_2.11-2.0.0.tgz
cd kafka_2.11-2.0.0
```

启动

```sh
# zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties
bin/kafka-server-start.sh config/server.properties
```


[kafka-docker](https://github.com/wurstmeister/kafka-docker)

kafka topic name: [a-zA-Z0-9\\\\._\\\\-], \".\"和\"_\"不能一起使用


Kafka Zookeeper ACL:

[Kafka Connect Security Basics](https://docs.confluent.io/platform/current/connect/security.html)
```properties

```

## security

[Kerberos](http://web.mit.edu/kerberos/)
[Kerberos 维基百科](https://zh.wikipedia.org/wiki/Kerberos)

参考资料: [利用 kerberos 提供票據加密](http://linux.vbird.org/linux_enterprise/kerberos.php)

## kafka-connector

(MongoDB Connector)[https://www.mongodb.com/kafka-connector]

kafka-log4j-appender
[text](https://www.devglan.com/apache-kafka/stream-log4j-logs-to-kafka)
[text](http://logging.apache.org/log4j/2.x/manual/appenders.html#KafkaAppender)

kafka connect elastic
[The Simplest Useful Kafka Connect Data Pipeline in the World…or Thereabouts – Part 2](https://www.confluent.io/blog/the-simplest-useful-kafka-connect-data-pipeline-in-the-world-or-thereabouts-part-2/)
[Kafka Connect Elasticsearch: Consuming and Indexing with Kafka Connect](https://sematext.com/blog/kafka-connect-elasticsearch-how-to/)
[Kafka Connect Elasticsearch Connector in Action](https://www.confluent.io/blog/kafka-elasticsearch-connector-tutorial/)


`PluginClassLoader` 加载 Plugin 类
`org.apache.kafka.connect.connector.Connector` 插件入口类


## kafka stream


[Tutorial: Write a Kafka Streams Application](https://kafka.apache.org/28/documentation/streams/tutorial)

## spark

## flink

## spring-kafka

1. spring kafka - Magic v1 not supporting record headers 错误


rg.springframework.kafka.support.serializer.JsonSerializer

spring.json.add.type.headers

`ConsumerSeekAware` 监听 kafka seek 操作, 使用 `@KafkaListener` 注解的类如果实现该接口, `org.springframework.kafka.annotation.KafkaListenerAnnotationBeanPostProcessor#processListener` 把该bean注入到 `org.springframework.kafka.listener.adapter.MessagingMessageListenerAdapter` 实例中。如果应用想要操作seek可继承 `org.springframework.kafka.listener.AbstractConsumerSeekAware`实现 `seekToBeginning` 等功能

`AbstractKafkaListenerContainerFactory` 创建消息容器

`MethodKafkaListenerEndpoint` 处理 `KafkaListener` 注解标注的方法

从"kafka"中读取消息时，配置最长等待时间和最小获取消息数来批量读取消息，提高数据写入效率。如果本批次出现错误，则保存批次号，时间，相关错误数据等到错误消息队列，以便分析处理重新入队。具体实现可参考"kafka JDBC connector"提高程序稳定性，参考类:

* `org.apache.kafka.connect.sink.SinkRecord` 记录消息批次，偏移量，时间戳，消息体等
* `org.apache.kafka.connect.runtime.WorkerSinkTask` 拉取消息任务

Spring Kafka:

ConcurrentMessageListenerContainer 内部创建多个 KafkaMessageListenerContainer

`org.springframework.kafka.listener.KafkaMessageListenerContainer.ListenerConsumer#run` 负责执行具体的"poll"操作

```java
protected void pollAndInvoke() {
    // 框架进行提交操作, 2.3 以后默认关闭自动提交
    // 关闭自动提交, AckMode 不是 RECORD, 默认为 AckMode.BATCH
    if (!this.autoCommit && !this.isRecordAck) {
        processCommits();
    }
    fixTxOffsetsIfNeeded();
    idleBetweenPollIfNecessary();
    ...
}
```

在调用listener失败时, 默认的批量信息异常处理器为`org.springframework.kafka.listener.RecoveringBatchErrorHandler`, 当异常为 `BatchListenerFailedException` 且包含错误消息时尝试提交正确的消息偏移量, 如果其他情况则调用 `SeekToCurrentBatchErrorHandler` 把该组的偏移量进行seek到错误数据之前的位置, 并抛出异常

org.springframework.boot.autoconfigure.kafka.KafkaAnnotationDrivenConfiguration

`org.springframework.kafka.listener.KafkaMessageListenerContainer#publishConsumerStoppedEvent` 关闭应用时发布相关事件

Kafka Server:

`kafka.server.KafkaApis#handleFetchRequest` 处理 fetch 请求（数据下载），
`kafka.server.Defaults` 定义配置项

问题, 当无新消息到达时打印如下日志:

> o.a.kafka.clients.FetchSessionHandler    : [Consumer clientId=consumer-app-data-engine-1, groupId=app-data-engine] Error sending fetch request (sessionId=INVALID, epoch=INITIAL) to node 0:
 org.apache.kafka.common.errors.DisconnectException: null

 Error registering AppInfo mbean

 ## 常用命令:

 1. 组列表: ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
 1. 主题列表: ./kafka-topics.sh --bootstrap-server localhost:9092 --list
 3. 修改消费进度: ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --topic xxx_topic  --group xxx_group --reset-offsets --to-latest --execute

 ## 其它

### 1. Kafka 3.0

 Kafka 3.0 推出，基本上是常规升级，api 优化等，另外还有 “A preview of KRaft mode is available“， “The producer should enable the strongest message delivery guarantee by default.”

 * [KIP-679: Producer will enable the strongest delivery guarantee by default](https://cwiki.apache.org/confluence/display/KAFKA/KIP-679%3A+Producer+will+enable+the+strongest+delivery+guarantee+by+default)
 * [KIP-98 - Exactly Once Delivery and Transactional Messaging](https://cwiki.apache.org/confluence/display/KAFKA/KIP-98+-+Exactly+Once+Delivery+and+Transactional+Messaging)

 ### 2. Kafka Tools

 [System Tools](https://cwiki.apache.org/confluence/display/KAFKA/System+Tools)

* Consumer Offset Checker
* Dump Log Segment
* Export Zookeeper Offsets
* Get Offset Shell
* Import Zookeeper Offsets
* JMX Tool
* Kafka Migration Tool
* Mirror Maker
* Replay Log Producer
* Simple Consumer Shell
* State Change Log Merger
* Update Offsets In Zookeeper
* Verify Consumer Rebalance

### 参考资料

 * [message format](https://kafka.apache.org/documentation/#messages)