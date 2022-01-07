
spring actuator 暴露监控相关信息接口（JVM以及数据库连接信息等），可 export 到 promethues，然后用 grafana 展示

健康状态检测：

* org.springframework.boot.actuate.elasticsearch.ElasticsearchReactiveHealthIndicator
* org.springframework.boot.actuate.elasticsearch.ElasticsearchRestHealthIndicator


```java
// org.springframework.data.elasticsearch.client.reactive.DefaultReactiveElasticsearchClient#execute
public Mono<ClientResponse> execute(ReactiveElasticsearchClientCallback callback) {

    return this.hostProvider.getActive(Verification.LAZY) //
            .flatMap(callback::doWithClient) //
            .onErrorResume(throwable -> {

                if (throwable instanceof ConnectException) {

                    return hostProvider.getActive(Verification.ACTIVE) //
                            .flatMap(callback::doWithClient);
                }

                return Mono.error(throwable);
            });
}
```

避免 `ElasticSearchReactiveHealthContributorAutoConfiguration` 先创建， 但是需要 `ElasticSearchRestHealthContributorAutoConfiguration`，在应用中排除 `ElasticSearchReactiveHealthContributorAutoConfiguration.class` 即可

actuator

org.springframework.boot.actuate.autoconfigure.endpoint.web.servlet.WebMvcEndpointManagementContextConfiguration 自动配置 org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping

```java
// org.springframework.boot.actuate.autoconfigure.endpoint.web.servlet.WebMvcEndpointManagementContextConfiguration
// org.springframework.boot.actuate.autoconfigure.health.HealthEndpointAutoConfiguration

// org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping
// 处理 base path 下的监控信息请求
// management.endpoints.web.base-path=/xxxx
```

`MappingJackson2HttpMessageConverter` 支持的 MediaType 为 `new MediaType("application", "*+json")`, 即也支持 "Content-Type" 为 "application/vnd.spring-boot.actuator.v2+json" 等所有 JSON 类型, 具体判断逻辑可参考 `org.springframework.util.MimeType#includes`

org.springframework.boot.actuate.metrics.MetricsEndpoint 暴露 metrics, io.micrometer.core.instrument.binder 包下的类实现了常见的监控指标, io.micrometer.core.instrument.binder.MeterBinder 接口定义数据如何绑定到 MeterRegistry 中, org.springframework.boot.actuate.autoconfigure.metrics.MeterRegistryPostProcessor 通过 org.springframework.boot.actuate.autoconfigure.metrics.MeterRegistryConfigurer 把 binder， filter 等绑定注入到 MeterRegistry 中


<!-- 存储统计数据到  elastic-->
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-elastic</artifactId>
  <version>${micrometer.version}</version>
</dependency>

<!-- 暴露数据到  prometheus-->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>${micrometer.version}</version>
</dependency>
