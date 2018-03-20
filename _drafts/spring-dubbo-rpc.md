# Spring Boot + Dubbo 实现RPC

安装Zookeeper：
```sh
brew install zookeeper
zkServer start
```
java:

pom.xml
```xml

<properties>
    <spring-boot.version>1.5.10.RELEASE</spring-boot.version>
    <dubbo.version>2.6.0</dubbo.version>
    <zookeeper.version>3.4.9</zookeeper.version>
    <curator.version>0.2</curator.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>${spring-boot.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <version>${spring-boot.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>${dubbo.version}</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.springframework</groupId>
                <artifactId>spring</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>${zookeeper.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>${curator.version}</version>
    </dependency>
</dependencies>
```

提供者：

```java
@SpringBootApplication(scanBasePackages = "net.cofcool.test.controller")
@DubboComponentScan(basePackages = "net.cofcool.test.server.service")
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public ApplicationConfig dubboApplicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("provider");

        return applicationConfig;
    }

    @Bean
    public ProtocolConfig protocolConfig() {
        ProtocolConfig protocol = new ProtocolConfig();
        protocol.setName("rmi");
        protocol.setPort(12345);
        protocol.setThreads(200);

        return protocol;
    }

    @Bean
    public RegistryConfig registryConfig() {
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setAddress("zookeeper://127.0.0.1:2181");
        registryConfig.setClient("curator");

        return registryConfig;
    }


}
```
service:
```java
@Service(timeout = 5000)
public class SmschkServiceImpl implements SmschkService<Smschk> {
    ...
}
```

消费者:
```java
@SpringBootApplication(scanBasePackages = "net.cofcool.controller")
@DubboComponentScan(basePackages = "net.cofcool.controller")
public class ClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ClientApplication.class, args);
    }

    @Bean
    public ApplicationConfig dubboApplicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("sms-customer");

        return applicationConfig;
    }

    @Bean
    public EmbeddedServletContainerFactory embeddedServletContainerFactory() {
        TomcatEmbeddedServletContainerFactory containerFactory = new TomcatEmbeddedServletContainerFactory();
        containerFactory.setPort(8888);

        return containerFactory;
    }

    @Bean
    public ProtocolConfig protocolConfig() {
        ProtocolConfig protocol = new ProtocolConfig();
        protocol.setName("rmi");
        protocol.setPort(12345);
        protocol.setThreads(200);

        return protocol;
    }

    @Bean
    public RegistryConfig registryConfig() {
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setAddress("zookeeper://127.0.0.1:2181");
        registryConfig.setClient("curator");

        return registryConfig;
    }


    @Bean
    public ConsumerConfig consumerConfig() {
        ConsumerConfig consumerConfig = new ConsumerConfig();
        consumerConfig.setTimeout(3000);

        return consumerConfig;
    }


}
```
```java
public class CustomerService {

    @Reference
    private SmschkService smschkService;

    ...
}
```
