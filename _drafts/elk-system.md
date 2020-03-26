# ELK 搭建

环境:

* Java
* Ubuntu server LTS 16.04
* Log4J

目录:



## 1. 简要介绍

## 2. 环境搭建

Java

## 3. 安装ELK

elasticsearch
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.2.tar.gz
tar -xzf elasticsearch-6.2.2.tar.gz
cd elasticsearch-6.2.2
./bin/elasticsearch



echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install elasticsearch


vim config/elasticsearch.yml

cluster.name=test
node.name=node-1
network.host=localhost
http.port=9200

v7

network.host=0.0.0.0
discovery.seed_hosts: ["127.0.0.1", "::1"]


$ curl localhost:9200
{
  "name" : "node-1",
  "cluster_name" : "my-application",
  "cluster_uuid" : "PVKDcod-Te6DkbxIjvtssw",
  "version" : {
    "number" : "6.2.2",
    "build_hash" : "10b1edd",
    "build_date" : "2018-02-16T19:01:30.685723Z",
    "build_snapshot" : false,
    "lucene_version" : "7.2.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

filebeat
```
# wget  https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.2.3-linux-x86_64.tar.gz
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.2.3-amd64.deb
sudo dpkg -i filebeat-6.2.3-amd64.deb

# configue
filebeat.prospectors:
- type: log
  enabled: true
  paths:
    - /var/log/*.log

# 只能启用一个output
# output.elasticsearch:
    # hosts: ["localhost:9200"]

output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5000"]

setup.kibana:
  host: "localhost:5601"

```

logstash
```
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.2.2.tar.gz
tar -xzf logstash-6.2.2.tar.gz
cd logstash-6.2.2

# install log4j plugin
./bin/logstash-plugin install logstash-input-log4j
# install beats plugin
./bin/logstash-plugin install logstash-input-beats

wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install logstash



vim config/log4j_to_es.conf

# For detail structure of this file
# Set: https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html
input {
  # For detail config for log4j as input,
  # See: https://www.elastic.co/guide/en/logstash/current/plugins-inputs-log4j.html
  log4j {
    mode => "server"
    host => "localhost"
    port => 4567
  }

  # use filebeat
  beats {
    port => 5000
  }
}

filter {
  # filter
}
output {
  # For detail config for elasticsearch as output,
  # See: https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html
  elasticsearch {
    action => "index"          #The operation on ES
    hosts  => "localhost:9200"   #ElasticSearch host, can be array.
    index  => "applog"         #The index to write data to.
  }
}

./bin/logstash -f config/log4j_to_es.conf


```

log4j
```
log4j.logger.com.demo.elk=DEBUG, socket

# appender socket
log4j.appender.socket=org.apache.log4j.net.SocketAppender
log4j.appender.socket.Port=4567
log4j.appender.socket.RemoteHost=localhost
log4j.appender.socket.layout=org.apache.log4j.PatternLayout
log4j.appender.socket.layout.ConversionPattern=%d [%-5p] [%l] %m%n
log4j.appender.socket.ReconnectionDelay=10000
```

kibana
```
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.2.2-linux-x86_64.tar.gz
tar -xzf kibana-6.2.2-linux-x86_64.tar.gz
cd kibana-6.2.2-linux-x86_64

vim config/kibana.yml

./bin/kibana

http://localhost:5601
```
