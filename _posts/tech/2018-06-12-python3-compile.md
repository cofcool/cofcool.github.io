---
layout: post
category: Tech
title: 从源码编译安装Python3
tags: [python]
excerpt: Centos6通过编译源码的方式安装Python3。
---

{% include JB/setup %}

基础环境:

* Centos 6

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=2 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 下载](#1-下载)
* [2. 编译配置](#2-编译配置)
* [3. 安装](#3-安装)

<!-- /code_chunk_output -->


## 1. 下载

```sh
# 安装依赖和基础环境
yum install gcc zlib-devel make openssl-devel
wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz
```

## 2. 编译配置

启用SSL模块

编辑`Modules/Setup.dist`，取消以下行的注释。

```
_socket socketmodule.c

# Socket module helper for SSL support; you must comment out the other
# socket line above, and possibly edit the SSL variable:
# SSL=/usr/local/ssl
_ssl _ssl.c \
 -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
 -L$(SSL)/lib -lssl -lcrypto
```

指定安装位置为`/opt/python3`。

```sh
./configure --prefix=/opt/python3
```

## 3. 安装

```sh
make && make install
```

链接到系统执行文件目录:
```sh
ln -s /opt/python3/bin/python3 /usr/bin/python3
ln -s /opt/python3/bin/pip3 /usr/bin/pip3
```
