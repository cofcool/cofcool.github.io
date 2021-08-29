
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

```java
// org.springframework.boot.actuate.autoconfigure.endpoint.web.servlet.WebMvcEndpointManagementContextConfiguration
// org.springframework.boot.actuate.autoconfigure.health.HealthEndpointAutoConfiguration

// org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping
// 处理 base path 下的监控信息请求
// management.endpoints.web.base-path=/xxxx
```