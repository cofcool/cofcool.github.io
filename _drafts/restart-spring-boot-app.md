# Spring Boot 项目重启

## 1. 

```java
@SpringBootApplication
public class Application {

    private static ConfigurableApplicationContext context;

    public static void main(String[] args) {
        context = SpringApplication.run(Application.class, args);
    }

    public static void restart() {
        ApplicationArguments args = context.getBean(ApplicationArguments.class);

        Thread thread = new Thread(() -> {
            context.close();
            context = SpringApplication.run(Application.class, args.getSourceArgs());
        });

        thread.setDaemon(false);
        thread.start();
    }
}
```

## 2. RestartEndpoint

spring-boot-starter-actuator

## 3. Spring Cloud RestartService

## 注意

ApplicationContext.refresh()  `GenericApplicationContext does not support multiple refresh attempts`
