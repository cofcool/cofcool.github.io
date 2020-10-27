# prometheus 

## 安装

参考官方文档 https://prometheus.io/docs/prometheus/latest/installation/，推荐使用 docker 方式运行。

```sh
docker run \
    -p 9090:9090 \
    -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```


`Grafana`, 下载地址 https://grafana.com/grafana/download?platform=docker，docker 运行:

```sh
docker run -d --name=grafana -p 3000:3000 grafana/grafana:7.1.5-ubuntu
```

## 配置

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

## 采集

### 采集节点信息

```sh
wget https://github.com/prometheus/node_exporter/releases/download/v*/node_exporter-*.*-amd64.tar.gz
tar xvfz node_exporter-*.*-amd64.tar.gz
cd node_exporter-*.*-amd64
./node_exporter
```

```yml
scrape_configs:
  - job_name: node
    static_configs:
      - targets: ['localhost:9100']
```

dashboard 模版:

JVM (Micrometer) dashboard
https://grafana.com/grafana/dashboards/4701

node export
https://grafana.com/grafana/dashboards/1860

### Spring 应用

依赖:

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

配置文件:

```properties
management.endpoint.metrics.enabled=true
management.endpoints.web.exposure.include=*
management.endpoint.prometheus.enabled=true
management.metrics.export.prometheus.enabled=true
management.metrics.export.prometheus.step=1
```

示例代码:

```java
public class Application extends SpringBootServletInitializer {

    ...

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

访问"/actuator/demo"即可获取信息，