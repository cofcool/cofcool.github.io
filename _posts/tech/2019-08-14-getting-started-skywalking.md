---
layout: post
category: Tech
title: 简单使用 SkyWalking
tags: [notes]
excerpt: SkyWalking 是一个分布式系统的应用程序性能监视工具，专为微服务、云原生架构和基于容器（Docker、K8s、Mesos）架构而设计
---

{% include JB/setup %}

[SkyWalking](https://skywalking.apache.org/) 是为微服务等分布式架构设计, 可以很简易的部署和使用。

> 分布式系统的应用程序性能监视工具，专为微服务、云原生架构和基于容器（Docker、K8s、Mesos）架构而设计。

![skywalking-frame](https://skywalking.apache.org/images/home/architecture_2160x720.png?t=20220617)

`OPA` 服务负责分析处理从 `Agent` 客户端上报上来的数据, 然后由 `UI` 呈现出来。

官方 [参考文档](https://github.com/apache/skywalking/blob/master/docs/en/setup/README.md), 本文以单机 Java 服务为例(JDK 8)。

## 1.安装

[下载地址](https://skywalking.apache.org/downloads/), 下载解压即可。

## 2. 使用

**Backend and UI**:
```
# 启动 OAP 服务和 UI 程序
./bin/startup.sh 

# OAP 服务配置文件, 可配置监听端口, grpc等
config/application.yml

# UI 配置文件, 可配置访问端口, 连接 OAP 服务等
webapp/webapp.yml
```

**Agent**:
```
# 启用 agent
java -javaagent:/path/to/skywalking-agent/skywalking-agent.jar -jar yourApp.jar

# 配置文件, 配置服务名称, 连接 OAP 端口等
agent/config/agent.config
```

以上步骤完成后即可通过浏览器访问, 默认端口为`8080`。