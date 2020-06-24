---
layout: post
category : Tech
title : 简单使用 RabbitMQ
tags : [ops, java, notes]
excerpt: RabbitMQ 是实现了 AMQP 的开源消息代理软件，使用 Erlang 开发，常用于分布式系统中的消息存储转发等，在易用性、扩展性、高可用性等方面都非常的优秀
---
{% include JB/setup %}

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 安装](#1-安装)
  - [1.1 手动安装](#11-手动安装)
  - [1.2 Docker 镜像](#12-docker-镜像)
- [2. 配置](#2-配置)
- [3. Spring Boot 项目集成](#3-spring-boot-项目集成)
  - [1. AmqpTemplate](#1-amqptemplate)
  - [2. RabbitListener](#2-rabbitlistener)
  - [3. AmqpAdmin](#3-amqpadmin)
- [4. 常见问题](#4-常见问题)
  - [4.1 RabbitMQ 不同版本安装问题](#41-rabbitmq-不同版本安装问题)

<!-- /code_chunk_output -->

环境:

* Ubuntu 18.04
* RabbitMQ v3.7.x
* Spring Boot v2.2.2.RELEASE

`RabbitMQ` 是实现了 `AMQP` 的开源消息代理软件，使用 `Erlang` 开发，常用于分布式系统中的消息存储转发等，在易用性、扩展性、高可用性等方面都非常的优秀。 

### 1. 安装

#### 1.1 手动安装

安装参考文档 [Installing on Debian and Ubuntu ](https://www.rabbitmq.com/install-debian.html)

```sh
#!/bin/sh

## If sudo is not available on the system,
## uncomment the line below to install it
# apt-get install -y sudo

sudo apt-get update -y

## Install prerequisites
sudo apt-get install curl gnupg -y

## Install RabbitMQ signing key
curl -fsSL https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc | sudo apt-key add -

## Install apt HTTPS transport
sudo apt-get install apt-transport-https

sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list <<EOF 
## Installs the latest Erlang 22.x release.
## Change component to "erlang-21.x" to install the latest 21.x version.
## "bionic" as distribution name should work for any later Ubuntu or Debian release.
## See the release to distribution mapping table in RabbitMQ doc guides to learn more.
deb https://dl.bintray.com/rabbitmq-erlang/debian bionic erlang
deb https://dl.bintray.com/rabbitmq/debian bionic main
EOF


## Update package indices
sudo apt-get update -y

## Install rabbitmq-server and its dependencies
sudo apt-get install rabbitmq-server -y --fix-missing
```

运行:

```sh
sudo rabbitmq-server start
```

#### 1.2 Docker 镜像

拉取镜像([rabbitmq image](https://registry.hub.docker.com/_/rabbitmq/)):

```sh
# 镜像基于 Ubuntu 18.04，版本为 3.7.x，已启用 rabbitmq_management 插件
docker pull rabbitmq:3.7-management
```

运行:

```sh
docker run -it --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.7-management
```

默认账户密码为"guest-guest"，如果需要配置用户名密码可在运行时添加参数: `-e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=password`

### 2. 配置

官方文档为 [Configuration](https://www.rabbitmq.com/configure.html)

示例配置文件为 [rabbitmq.conf.example](https://raw.githubusercontent.com/rabbitmq/rabbitmq-server/master/docs/rabbitmq.conf.example)

添加用户：

```sh

sudo rabbitmqctl add_user cofcool
sudo rabbitmqctl set_permissions --vhost / cofcool '.*' '.*' '.*'
```

启用 `rabbitmq_management` 插件：

```sh
sudo rabbitmq-plugins enable rabbitmq_management
```

### 3. Spring Boot 项目集成

官方文档为 [Spring AMQP](https://docs.spring.io/spring-amqp/docs/2.2.2.RELEASE/reference/html)

添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

基本配置项：

```properties
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.virtual-host=/
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

#### 1. AmqpTemplate

该类定义了 `AMQP` 的基本方法，如消息发送接收等，部分方法如下：

发送消息:

```java
void send(Message message) throws AmqpException;

void send(String routingKey, Message message) throws AmqpException;

void send(String exchange, String routingKey, Message message) throws AmqpException;
```

接收消息:

```java
Message receive() throws AmqpException;

Message receive(String queueName) throws AmqpException;

Message receive(long timeoutMillis) throws AmqpException;

Message receive(String queueName, long timeoutMillis) throws AmqpException;
```

更多内容参考 [AmqpTemplate](https://docs.spring.io/spring-amqp/docs/2.2.2.RELEASE/reference/html/#amqp-template)

#### 2. RabbitListener

简单示例如下，使用注解 `@RabbitListener` 监听消息事件：

```java
@RabbitListener(queues = QUEUE_KEY)
public void listen(String in) {
    log.info("received: {}", in);
}
```

更多内容参考 [Annotation-driven Listener Endpoints](https://docs.spring.io/spring-amqp/docs/2.2.2.RELEASE/reference/html/#async-annotation-driven)

#### 3. AmqpAdmin

该类定义了 `AMQP` 的相关管理操作，如创建队列等，部分定义如下：

```java
void declareExchange(Exchange exchange);

boolean deleteExchange(String exchangeName);

String declareQueue(Queue queue);

boolean deleteQueue(String queueName);
```

### 4. 常见问题

#### 4.1 RabbitMQ 不同版本安装问题

本文开始写时 RabbitMQ 的版本为`3.7.x`，但是一直拖延等到下定决心要完成时发现已经更新到了`3.8.x`，虽然安装方法基本一致，但是稍有不同，请参考 [Quick Start Example](https://www.rabbitmq.com/install-debian.html#apt-bintray-quick-start)