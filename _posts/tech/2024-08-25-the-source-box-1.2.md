---
layout: post
category : Tech
title : 秘源宝箱发布 v1.2
tags : [my, software]
excerpt: 秘源宝箱发布 v1.2, converts 新增字节单位转换更易读....
---
{% include JB/setup %}

下载地址: [Release 1.2.0 · cofcool/sourcebox](https://github.com/cofcool/sourcebox/releases/tag/1.2.0)

秘源宝箱 v1.2 变更：

**新增**

* 增加 `task`, 重复执行 shell 命令, 支持执行次数, 执行频率
* 增加 `NetworkUtils`, IP 的 DNS 纪录解析, 读取 IP 信息
* 增加 `DiffAnalysis`, 根据 diff 信息抽取对应行的变更记录
* 增加 `FileTools`, 文件处理工具

**增强**

* `converts` 新增: 字节单位转换更易读, 字符串加解密, 生成摩尔斯电码
* `json2POJO` 支持生成 Kotlin 代码
* `json` 支持格式化 jsonline
* 部分功能使用 Go 开发