---
layout: post
category : Tech
title : Spring MVC 单元测试
tags : [java]
excerpt: 在开发中，测试的重要性不言而喻。今天我们来看看，Spring MVC如何进行单元测试。
---
{% include JB/setup %}

在开发中，测试的重要性不言而喻。今天我们来看看，Spring MVC如何进行单元测试。

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 依赖配置](#1-依赖配置)
- [2. 编写测试代码](#2-编写测试代码)
- [3. 常见问题](#3-常见问题)
  - [3.1 项目集成Swagger](#31-项目集成swagger)
  - [3.2 添加数据源](#32-添加数据源)
  - [3.3 使用Hibernate Validator](#33-使用hibernate-validator)
  - [3.4 Junit版本](#34-junit版本)
  - [3.5 执行Maven打包时跳过测试](#35-执行maven打包时跳过测试)

<!-- /code_chunk_output -->



### 1. 依赖配置

```xml
<!--> maven 依赖 <-->
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>
    <!-- spring mvc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- spring boot -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
...
<build>
  <!--> 测试需要的资源文件 <-->
  <testResources>
    <testResource>
        <directory>src/test/resources</directory>
    </testResource>
  </testResources>
</build>
```

### 2. 编写测试代码

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:spring.xml"})
@WebAppConfiguration
public class SpringTest {

    @Resource
    private TestService testService;

    @Test
    public void testQuery() {
        Assert.assertNotNull(testService.query());
    }
}
```

如果想要一次运行多个测试类，可参考以下示例。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:spring.xml"})
@WebAppConfiguration
@SuiteClasses({SpringTest.class})
public class TestAll {

}
```

Spring Boot:

```java
// junt 5, spring-boot-test 2.4
@SpringBootTest
@ExtendWith(SpringExtension.class)
// MockMvc
@AutoConfigureMockMvc
// web mvc
@AutoConfigureWebMvc
// JPA
@AutoConfigureDataJpa
@EnableJpaRepositories
@EntityScan
```

### 3. 常见问题

#### 3.1 项目集成Swagger

如果使用`springfox-swagger2`，可能会抛出`Error creating bean with name 'documentationPluginsBootstrapper'`。原因是该项目使用`WebApplicationContext`，在测试类上加`@WebAppConfiguration`注解即可。

#### 3.2 添加数据源

```xml
<bean id="dataSource"
    class="org.springframework.jdbc.datasource.DriverManagerDataSource"
    p:driverClassName="com.mysql.jdbc.Driver"
    p:url="jdbc:mysql://${ADDRESS}"
    p:username="${USER}"
    p:password="${PASSWD}" />
```

#### 3.3 使用Hibernate Validator

若抛出缺少`javax.el`异常的话，添加如下依赖：

```xml
 <dependency>
     <groupId>org.glassfish</groupId>
     <artifactId>javax.el</artifactId>
     <version>${el-version}</version>
 </dependency>
```

#### 3.4 Junit版本

Spring 4.3要求4.12以上版本的的Junit。

```java
// 参考SpringJUnit4ClassRunner类
if (!ClassUtils.isPresent("org.junit.internal.Throwables", SpringJUnit4ClassRunner.class.getClassLoader())) {
  throw new IllegalStateException("SpringJUnit4ClassRunner requires JUnit 4.12 or higher.");
}
```

#### 3.5 执行Maven打包时跳过测试

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <configuration>
        <skip>true</skip>
      </configuration>
    </plugin>
  </plugins>
</build>
```
