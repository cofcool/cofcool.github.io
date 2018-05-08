---
layout: post
category : Tech
title : Gitkraken的简单使用
tags : [git]
excerpt: Gitkraken是一款Git的图形化管理工具，简单易用，支持常用的Git操作。
---
{% include JB/setup %}

这是一款Git的图形化管理工具，简单易用，支持常用的Git操作。

> [Gitkraken](https://www.gitkraken.com/), The legendary Git GUI client for Windows, Mac and Linux。

下面来简单介绍一下该软件的使用。

软件的整体界面如下:

* 左侧包含仓库地址，标签，子模块等
* 右侧界面为提交的log信息，包含提交者，修改文件等信息

![](http://cofcool.net/imgs/gitkraken-01.png)

下面我们来看看如何使用该软件。

**1**. 点击左上角的文件图标即可打开如下操作菜单，可打开本地仓库，clone远程仓库，创建本地仓库。

![](http://cofcool.net/imgs/gitkraken-02.png)

**2**. 如果有多个git配置信息，可取消勾选如下选项。

![](http://cofcool.net/imgs/gitkraken-03.png)

**3**. 更改软件默认使用的SSH Key，以及生成等。

![](http://cofcool.net/imgs/gitkraken-04.png)

**4**. 为了避免执行`Pull`的时候，与本地修改冲突，自动合并等，可把默认操作改为`Fetch`，避免出现以上问题。

![](http://cofcool.net/imgs/gitkraken-05.png)

**5**. 修改远程仓库地址，名称等。

![](http://cofcool.net/imgs/gitkraken-06.png)

**6**. 针对某次的提交进行操作，包括checkout，reset，添加tag等。

![](http://cofcool.net/imgs/gitkraken-07.png)
