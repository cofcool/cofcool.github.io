# prometheus 

## 安装

https://prometheus.io/docs/prometheus/latest/installation/

```sh
docker run \
    -p 9090:9090 \
    -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```

grafana, [Download Grafana](https://grafana.com/grafana/download?platform=docker)
```sh
docker run -d --name=grafana -p 3000:3000 grafana/grafana:7.1.5-ubuntu
```


```
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

  # Details to connect Prometheus with Spring Boot actuator end point to scrap the data
  # The job name is added as a label `job=` to any time series scraped from this config.
  - job_name: 'spring-actuator'
   
    # Actuator end point to collect the data. 
    metrics_path: '/actuator/prometheus'

    #How frequently to scape the data from the end point
    scrape_interval: 5s

    #target end point. We are using the Docker, so local host will not work. You can change it with
    #localhost if not using the Docker.
    static_configs:
    - targets: ['HOST_IP:8080']
```

```sh
docker run \
    -p 9090:9090 \
    -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```

---

wget https://github.com/prometheus/node_exporter/releases/download/v*/node_exporter-*.*-amd64.tar.gz
tar xvfz node_exporter-*.*-amd64.tar.gz
cd node_exporter-*.*-amd64
./node_exporter


scrape_configs:
- job_name: node
  static_configs:
  - targets: ['localhost:9100']
---

JVM (Micrometer) dashboard
https://grafana.com/grafana/dashboards/4701

node export
https://grafana.com/grafana/dashboards/1860

## 集成

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

```properties
management.endpoint.metrics.enabled=true
management.endpoints.web.exposure.include=*
management.endpoint.prometheus.enabled=true
management.metrics.export.prometheus.enabled=true
management.metrics.export.prometheus.step=1
```

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