---
layout: post
category : Tech
title : iTerm2 导入配置文件
tags : [Mac,技巧]
excerpt: iTerm2 如何导入配置文件
---

{% include JB/setup %}

iTerm 是 macOS 上常用的一大利器，相比于系统自带终端功能更为强大易用，属于装机必备软件。

最近重装了 macOS，原打算直接把原系统的配置文件拷贝到新系统。它的配置文件路径如下：`~/Library/Preferences/com.googlecode.iterm2.plist`，直接拷到新系统相同路径下，但是未生效，配置文件未被加载。

后来发现，原来官方已提供了导入配置文件的方法。

iTerm 会监控`~/Library/Application Support/iTerm2/DynamicProfiles`路径，如果发现该路径下有新的文件，且符合要求即会加载为配置文件，文件格式最好为`Apple Property Lists`，也支持`JSON`等格式。

因此只需把原来的配置文件`com.googlecode.iterm2.plist`拷贝到指定目录即可。加载成功后可点击菜单 `Preferences -> Profiles` 找到加载的配置文件。

参考链接:

* [documentation-dynamic-profiles](https://www.iterm2.com/documentation-dynamic-profiles.html)