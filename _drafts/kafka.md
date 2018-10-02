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
