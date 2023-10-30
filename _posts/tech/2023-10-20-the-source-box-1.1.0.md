---
layout: post
category : Tech
title : 秘源宝箱发布 v1.1.0
tags : [my, software]
excerpt: 秘源宝箱发布 v1.1.0，新增网页下载，命令助手等功能
---
{% include JB/setup %}

秘源宝箱 v1.1.0 新增

* 命令助手
* 网页下载

## 命令助手

使用前需要把 `source ～/.mytool/alias` 添加到 shell 配置中

* 命令管理，方便管理日常使用的长命令，如查询，添加标签等
* 别名管理，可针对长命令配置别名，并导出到当前环境

使用:

* 添加 `./mytool.sh --tool=cHelper --add="@helper mytool --tool=cHelper #mytool #my"`，@开头的为别名，可选参数；#开头的为标签，可以为多个，可选参数
* 导出到环境变量 `./mytool.sh --tool=cHelper --store=ALL` ，--store 参数可指定需要导出的命令，支持别名和标签，如 `--store="#kafka"`，只会导出有别名的命令
* 查询 `./mytool.sh --tool=cHelper --find=ALL` 查询命令，支持别名和标签，可以多个，如 `--find="#my @helper"`
* 删除 `./mytool.sh --tool=cHelper --del=ALL` 删除命令，支持别名和标签，可以多个，如 `--del="#my @helper"`

## 网页下载

* 链接递归遍历
* 代理
* 批量下载
* 转换为 markdown 、text 、epub

使用: `./mytool.sh --tool=htmlDown --url="https://example.com"`

更多参考 https://github.com/cofcool/my-toolbox 