---
layout: post
category : Tech
title : Ubuntu Server 网络配置
tags : [ubuntu]
---
{% include JB/setup %}

目录：
<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 静态IP配置](#1-静态ip配置)
* [2. DNS设置](#2-dns设置)

<!-- /code_chunk_output -->



环境:

* Ubuntu Server 16.04

## 1. 静态IP配置

```shell
vim /etc/network/interfaces
# content
auto eth0  # 网卡名字
iface eth0 inet static # 静态IP
address 192.168.1.100  # 设置IP
netmask 255.255.255.0 # 子网掩码
gateway 192.168.1.1 # 网关
broadcast 192.168.1.255 # 广播地址
```

**注意**：虚拟机配置网卡时的网关必须是存在的，根据自己的IP段填写。

## 2. DNS设置

```shell
sudo vim /etc/resolvconf/resolv.conf.d/base
# content
nameserver 192.168.1.1 # 根据自己的路由IP
nameserver xxx.xxx.xxx.xxx
```
