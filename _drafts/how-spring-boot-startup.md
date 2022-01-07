# 简述 Spring Boot 应用启动行为


### ServletContextInitializer

ServletRegistrationBean, FilterRegistrationBean, DelegatingFilterProxyRegistrationBean, ServletListenerRegistrationBean 为子类
 
注册流程

org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#getServletContextInitializerBeans
org.springframework.boot.web.servlet.ServletContextInitializerBeans
org.springframework.boot.web.servlet.server.ServletWebServerFactory#getWebServer