---
layout: post
category : Tech
title : 快速搭建日志系统
tags : [ops]
excerpt: 大量服务部署时及时了解服务运行状态对整个系统来说至关重要, 因此需要一套具备日志收集、清洗、展示功能的日志系统
---
{% include JB/setup %}

环境:

* JDK 11
* Ubuntu server LTS 18.04

目录:


<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 简要介绍](#1-简要介绍)
- [2. 安装](#2-安装)
  - [2.1 Docker](#21-docker)
  - [2.2 安装包](#22-安装包)
- [3. 配置](#3-配置)
  - [3.1 elasticsearch](#31-elasticsearch)
  - [3.2 kibana](#32-kibana)
  - [3.3 logstash](#33-logstash)
  - [3.4 filebeta](#34-filebeta)
  - [3.5 监控 Spring Boot 项目](#35-监控-spring-boot-项目)

<!-- /code_chunk_output -->


## 1. 简要介绍

大量服务部署时及时了解服务运行状态对整个系统来说至关重要, 为了实现此目的需要一套具备日志收集、清洗、展示功能的日志系统。目前业内广泛使用的方案为 "ELK", 它不仅基于开源组件搭建, 而且相关资料详细, 基于维护, 易用性、可二次开发等的目我们选用此解决方案。

基本架构如下:

<!-- 

```plantuml
box HOST
APPLICATION -> FILEBEAT: log files
end box
activate LOGSTASH
FILEBEAT -> LOGSTASH
LOGSTASH -> LOGSTASH: filter
deactivate
LOGSTASH -> ELASTICSEARCH: storage logs
ELASTICSEARCH -> KIBANA: show readable log
```
-->
<svg xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" contentscripttype="application/ecmascript" contentstyletype="text/css" height="276px" preserveAspectRatio="none" style="width:554px;height:276px;" version="1.1" viewBox="0 0 554 276" width="554px" zoomAndPan="magnify"><defs><filter height="300%" id="f1mq0ttua7clwn" width="300%" x="-1" y="-1"><feGaussianBlur result="blurOut" stdDeviation="2.0"></feGaussianBlur><feColorMatrix in="blurOut" result="blurOut2" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 .4 0"></feColorMatrix><feOffset dx="4.0" dy="4.0" in="blurOut2" result="blurOut3"></feOffset><feBlend in="SourceGraphic" in2="blurOut3" mode="normal"></feBlend></filter></defs><g><rect fill="#DDDDDD" height="264.9766" style="stroke:#A80036;stroke-width:1.0;" width="208" x="1" y="6"></rect><text fill="#000000" font-family="sans-serif" font-size="13" font-weight="bold" lengthAdjust="spacingAndGlyphs" textLength="36" x="87" y="19.4951">HOST</text><rect fill="#FFFFFF" filter="url(#f1mq0ttua7clwn)" height="127.0547" style="stroke:#A80036;stroke-width:1.0;" width="10" x="256.5" y="95.3125"></rect><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="58" x2="58" y1="62.9609" y2="231.3672"></line><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="163" x2="163" y1="62.9609" y2="231.3672"></line><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="261" x2="261" y1="62.9609" y2="231.3672"></line><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="383" x2="383" y1="62.9609" y2="231.3672"></line><line style="stroke:#A80036;stroke-width:1.0;stroke-dasharray:5.0,5.0;" x1="511.5" x2="511.5" y1="62.9609" y2="231.3672"></line><rect fill="#FEFECE" filter="url(#f1mq0ttua7clwn)" height="31.6094" style="stroke:#A80036;stroke-width:1.5;" width="103" x="5" y="26.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="89" x="12" y="47.8848">APPLICATION</text><rect fill="#FEFECE" filter="url(#f1mq0ttua7clwn)" height="31.6094" style="stroke:#A80036;stroke-width:1.5;" width="103" x="5" y="230.3672"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="89" x="12" y="251.9004">APPLICATION</text><rect fill="#FEFECE" filter="url(#f1mq0ttua7clwn)" height="31.6094" style="stroke:#A80036;stroke-width:1.5;" width="79" x="122" y="26.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="65" x="129" y="47.8848">FILEBEAT</text><rect fill="#FEFECE" filter="url(#f1mq0ttua7clwn)" height="31.6094" style="stroke:#A80036;stroke-width:1.5;" width="79" x="122" y="230.3672"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="65" x="129" y="251.9004">FILEBEAT</text><rect fill="#FEFECE" filter="url(#f1mq0ttua7clwn)" height="31.6094" style="stroke:#A80036;stroke-width:1.5;" width="89" x="215" y="26.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="75" x="222" y="47.8848">LOGSTASH</text><rect fill="#FEFECE" filter="url(#f1mq0ttua7clwn)" height="31.6094" style="stroke:#A80036;stroke-width:1.5;" width="89" x="215" y="230.3672"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="75" x="222" y="251.9004">LOGSTASH</text><rect fill="#FEFECE" filter="url(#f1mq0ttua7clwn)" height="31.6094" style="stroke:#A80036;stroke-width:1.5;" width="127" x="318" y="26.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="113" x="325" y="47.8848">ELASTICSEARCH</text><rect fill="#FEFECE" filter="url(#f1mq0ttua7clwn)" height="31.6094" style="stroke:#A80036;stroke-width:1.5;" width="127" x="318" y="230.3672"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="113" x="325" y="251.9004">ELASTICSEARCH</text><rect fill="#FEFECE" filter="url(#f1mq0ttua7clwn)" height="31.6094" style="stroke:#A80036;stroke-width:1.5;" width="62" x="478.5" y="26.3516"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="48" x="485.5" y="47.8848">KIBANA</text><rect fill="#FEFECE" filter="url(#f1mq0ttua7clwn)" height="31.6094" style="stroke:#A80036;stroke-width:1.5;" width="62" x="478.5" y="230.3672"></rect><text fill="#000000" font-family="sans-serif" font-size="14" lengthAdjust="spacingAndGlyphs" textLength="48" x="485.5" y="251.9004">KIBANA</text><rect fill="#FFFFFF" filter="url(#f1mq0ttua7clwn)" height="127.0547" style="stroke:#A80036;stroke-width:1.0;" width="10" x="256.5" y="95.3125"></rect><polygon fill="#A80036" points="151.5,91.3125,161.5,95.3125,151.5,99.3125,155.5,95.3125" style="stroke:#A80036;stroke-width:1.0;"></polygon><line style="stroke:#A80036;stroke-width:1.0;" x1="58.5" x2="157.5" y1="95.3125" y2="95.3125"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="44" x="65.5" y="90.4561">log files</text><polygon fill="#A80036" points="244.5,105.3125,254.5,109.3125,244.5,113.3125,248.5,109.3125" style="stroke:#A80036;stroke-width:1.0;"></polygon><line style="stroke:#A80036;stroke-width:1.0;" x1="163.5" x2="250.5" y1="109.3125" y2="109.3125"></line><line style="stroke:#A80036;stroke-width:1.0;" x1="266.5" x2="308.5" y1="144.6641" y2="144.6641"></line><line style="stroke:#A80036;stroke-width:1.0;" x1="308.5" x2="308.5" y1="144.6641" y2="157.6641"></line><line style="stroke:#A80036;stroke-width:1.0;" x1="261.5" x2="308.5" y1="157.6641" y2="157.6641"></line><polygon fill="#A80036" points="271.5,153.6641,261.5,157.6641,271.5,161.6641,267.5,157.6641" style="stroke:#A80036;stroke-width:1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="24" x="273.5" y="139.8076">filter</text><polygon fill="#A80036" points="371.5,179.0156,381.5,183.0156,371.5,187.0156,375.5,183.0156" style="stroke:#A80036;stroke-width:1.0;"></polygon><line style="stroke:#A80036;stroke-width:1.0;" x1="266.5" x2="377.5" y1="183.0156" y2="183.0156"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="71" x="273.5" y="178.1592">storage logs</text><polygon fill="#A80036" points="499.5,209.3672,509.5,213.3672,499.5,217.3672,503.5,213.3672" style="stroke:#A80036;stroke-width:1.0;"></polygon><line style="stroke:#A80036;stroke-width:1.0;" x1="383.5" x2="505.5" y1="213.3672" y2="213.3672"></line><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacingAndGlyphs" textLength="104" x="390.5" y="208.5107">show readable log</text></g></svg>

## 2. 安装

### 2.1 Docker

```shell
docker pull elastic/logstash:7.10.1
docker pull elastic/kibana:7.10.1
docker pull elastic/filebeta:7.10.1
docker pull elastic/elasticsearch:7.10.1
docker pull grafana/grafana:7.3.6
```

参考文档：

* [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)
* [Install Kibana with Docker](https://www.elastic.co/guide/en/kibana/current/docker.html)
* [Running Logstash on Docker](https://www.elastic.co/guide/en/logstash/current/docker.html)
* [Run Filebeat on Docker](https://www.elastic.co/guide/en/beats/filebeat/current/running-on-docker.html)

### 2.2 安装包

官网下载安装包安装, 下载页面为 [Download Elastic Products](https://www.elastic.co/downloads/)

**Elasticsearch**

```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.1-linux-x86_64.tar.gz
tar -xzf elasticsearch-7.10.1-linux-x86_64.tar.gz
cd elasticsearch-7.10.1-linux-x86_64
./bin/elasticsearch

# apt
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install elasticsearch

# 验证是否安装成功
curl localhost:9200
```

**filebeat**

```shell
# wget  https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.10.1-linux-x86_64.tar.gz
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.10.1-amd64.deb
sudo dpkg -i filebeat-7.10.1-amd64.deb
```

**logstash**

```shell
wget https://artifacts.elastic.co/downloads/logstash/logstash-7.10.1-linux-x86_64.tar.gz
tar -xzf logstash-7.10.1-linux-x86_64.tar.gz
cd logstash-7.10.1-linux-x86_64

# install beats plugin
./bin/logstash-plugin install logstash-input-beats

# apt 
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install logstash
```

**kibana**

```shell
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.10.1-linux-x86_64.tar.gz
tar -xzf kibana-7.10.1-linux-x86_64.tar.gz
cd kibana-7.10.1-linux-x86_64
./bin/kibana
```

"kibana" 默认端口为 "5601", 打开浏览器访问即可。

## 3. 配置

### 3.1 elasticsearch

单节点运行：

```shell
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node"  elastic/elasticsearch:7.10.1
```

详细配置可通过镜像内部的 `/usr/share/elasticsearch/config` 目录进行配置, 也可通过文件挂载的方式挂载配置文件（推荐）。

允许本机之外设备访问:

```shell
# elasticsearch.yml
network.host=0.0.0.0
discovery.seed_hosts: ["127.0.0.1", "::1"]
```

### 3.2 kibana

连接 “elasticsearch” 节点并运行：


```
docker run --link YOUR_ELASTICSEARCH_CONTAINER_NAME_OR_ID:elasticsearch -p 5601:5601 elastic/kibana:7.10.1
```

配置文件路径为 `/opt/kibana/config/kibana.yml`, 可修改端口号, elasticsearch 地址等。

### 3.3 logstash

修改默认配置 `vi /opt/logstash/config/logstash.yml`:

```yaml
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "${ELASTICSEARCH_URI}" ]
```

Beats 插件配置参考 [Beats input plugin](https://www.elastic.co/guide/en/logstash/7.10/plugins-inputs-beats.html), `logstash.conf` 配置示例：

```yaml
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    # 2021-01-14 11:17:27.760 ERROR 16084 --- [freshExecutor-0] c.n.d.s.t.d.RedirectingEurekaHttpClient  : Request execution error
    match => { "message" => "(?m)^%{TIMESTAMP_ISO8601:timestamp}%{SPACE}%{LOGLEVEL:logLevel}%{SPACE}%{NUMBER:pid}%{SPACE}---%{SPACE}%{SYSLOG5424SD:threadName}%{SPACE}%{NOTSPACE:loggerName}%{SPACE}:%{SPACE}%{GREEDYDATA:message}" }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}

output {
  elasticsearch { 
    hosts => ["${ELASTICSEARCH_URI}"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

挂载该配置并运行:


```shell
docker run -p 5044:5044  -v ~/logstash.conf:/usr/share/logstash/pipeline/logstash.conf elastic/logstash:7.10.1
```

更多内容参考官方配置示例 [logstash-sample.conf](https://github.com/elastic/logstash/blob/master/config/logstash-sample.conf), Filter 配置参考 [Grok filter plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html), [Manage multiline messages](https://www.elastic.co/guide/en/beats/filebeat/7.10/multiline-examples.html)。

### 3.4 filebeta

配置日志输入和输出:

```yaml
filebeat.inputs:

# Each - is an input. Most options can be set at the input level, so
# you can use different inputs for various configurations.
# Below are the input specific configurations.

- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /var/logs/app/*
    
output.logstash:
  # The Logstash hosts
  hosts: ["${LOGSTASH_URI}"]
```

如果不使用 "Logstash" 直接输出到 "Elasticsearch" 并显示到 "Kibana" 配置如下:

```yaml
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["${ELASTICSEARCH_URI}"]
  
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  host: "${KIBANA_URI}"
```

直接输出到  "Elasticsearch"  需要使用模板功能, 执行如下启用改模板:

 ````shell
filebeta modules enable elasticsearch
filebeta setup
 ````

多行处理, 匹配以日期开头格式为"yyyy-MM-dd HH:mm:ss"的的行。
```yaml
- type: log
  multiline:
      pattern: '^\s*(\d{4})\-(\d{2})\-(\d{2})'
      negate: true
      match: after
      max_lines: 500
```

`Docker` 运行:

```shell
# 默认配置
docker run \
elastic/filebeat:7.10.1 \
setup -E setup.kibana.host=kibana:5601 \
-E output.elasticsearch.hosts=["elasticsearch:9200"]  

# 自定义配置
docker run -d \
  --name=filebeat \
  --user=root \
  --volume="$(pwd)/filebeat.docker.yml:/usr/share/filebeat/filebeat.yml:ro" \
  --volume="/var/lib/docker/containers:/var/lib/docker/containers:ro" \
  --volume="/var/run/docker.sock:/var/run/docker.sock:ro" \
  elastic/filebeat:7.10.1 filebeat -e -strict.perms=false \
  -E output.elasticsearch.hosts=["elasticsearch:9200"]  
```

### 3.5 监控 Spring Boot 项目

**Spring Boot 应用配置**

Maven 依赖:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

项目配置文件:

```properties
management.endpoint.metrics.enabled=true
management.endpoints.web.exposure.include=*
management.endpoint.prometheus.enabled=true
management.metrics.export.prometheus.enabled=true
management.metrics.export.prometheus.step=1
```

自定义数据监控示例代码:

```java
public class Application {

    @Configuration
    static class EndpointTest {

        @Bean
        public MessageEndpoint messageEndpoint() {
            return new MessageEndpoint();
        }

    }

    @WebEndpoint(id = "demo")
    static class MessageEndpoint {

        @ReadOperation(produces = "application/json")
        public Message<Void> message() {
            return Message.of("200", "Ok", null);
        }

    }
}
```

请求 `/actuator/demo` 路径如正确返回"OK"说明运行成功, 如想要导出信息到"Prometheus"修改路径即可。

**Prometheus**

安装参考官方文档 https://prometheus.io/docs/prometheus/latest/installation/, 推荐使用 docker 方式运行, 如下：

```sh
docker run \
    -p 9090:9090 \
    -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```


安装 `Grafana`, 下载地址 https://grafana.com/grafana/download?platform=docker, docker 运行:

```sh
docker run \
    -d --name=grafana \
    -p 3000:3000 \
    grafana/grafana:7.1.5-ubuntu
```

Prometheus 配置:

```yml
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'codelab-monitor'

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=` to any time series scraped from this config.
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']

  # 监控 spring boot 项目
  # spring actuator
  - job_name: 'spring-actuator'
   
    # 数据暴露点
    metrics_path: '/actuator/prometheus'

    # 每次采集间隔时间
    scrape_interval: 5s

    # 被监控的服务器地址
    static_configs:
      - targets: ['HOST_IP:8080']
```

以自定义配置的方式运行:

```sh
docker run \
    -p 9090:9090 \
    -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```

另外, 也可采集节点服务器信息, 安装 `node_exporter`(具体版本查看官方文档):

```sh
wget https://github.com/prometheus/node_exporter/releases/download/v*/node_exporter-*.*-amd64.tar.gz
tar xvfz node_exporter-*.*-amd64.tar.gz
cd node_exporter-*.*-amd64
./node_exporter
```

对应的 Prometheus 配置:

```yml
scrape_configs:
  - job_name: node
    static_configs:
      - targets: ['localhost:9100']
```

Grafana 提供的仪表板模版:

* [JVM (Micrometer) dashboard](https://grafana.com/grafana/dashboards/4701)
* [node export](https://grafana.com/grafana/dashboards/1860)
