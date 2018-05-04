# Tomcat笔记


<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. Tomcat结构解析](#1-tomcat结构解析)
  * [1.1 Tomcat服务器核心组件](#11-tomcat服务器核心组件)
* [资料引用](#资料引用)

<!-- /code_chunk_output -->


## 1. Tomcat结构解析

### 1.1 Tomcat服务器核心组件

组件层级如下所示：

* Server
  * Service
    * Connector
    * Engine
      * Host
        * Context

## 2. Session

### 2.1 Session实现类

1. StandardManager
2. PersistentManager
   1. FileStore
   2. JDBCStore

## 3. Http

### 3.1 HttpServlet

实现类为`org.apache.catalina.servletsDefaultServlet`

### 3.1 Request

实现类为`org.apache.catalina.connector.Request`

### 3.2 Response

实现类为`org.apache.catalina.connector.Response`

## 资料引用

1. 程序员突击:Tomcat原理与JavaWeb系统开发, 陈菁菁
