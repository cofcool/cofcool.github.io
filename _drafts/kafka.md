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

## kafka-connector

(MongoDB Connector)[https://www.mongodb.com/kafka-connector]

kafka-log4j-appender
[text](https://www.devglan.com/apache-kafka/stream-log4j-logs-to-kafka)
[text](http://logging.apache.org/log4j/2.x/manual/appenders.html#KafkaAppender)

kafka connect elastic
[The Simplest Useful Kafka Connect Data Pipeline in the World…or Thereabouts – Part 2](https://www.confluent.io/blog/the-simplest-useful-kafka-connect-data-pipeline-in-the-world-or-thereabouts-part-2/)
[Kafka Connect Elasticsearch: Consuming and Indexing with Kafka Connect](https://sematext.com/blog/kafka-connect-elasticsearch-how-to/)
[Kafka Connect Elasticsearch Connector in Action](https://www.confluent.io/blog/kafka-elasticsearch-connector-tutorial/)


## kafka stream

## spark

## flink

## spring-kafka

1. spring kafka - Magic v1 not supporting record headers 错误


rg.springframework.kafka.support.serializer.JsonSerializer

spring.json.add.type.headers