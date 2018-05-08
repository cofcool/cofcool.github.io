---
layout: post
category : Tech
title : Linux 自定义应用程序快捷方式
tags : [Linux]
excerpt: 一个简单文件即可在Linux上实现类似于Windows中的快捷方式。
---
{% include JB/setup %}

碰到一个新应用，想要体验一下的时候，却发现该程序只提供了tar包，每次打开程序都需要定位到程序目录，双击打开或是使用终端执行，烦不胜烦。为什么我们通常下载的`deb`格式的应用程序却不需要那么麻烦？

原来，使用`apt`或`dpkg`等安装的`deb`应用会在安装过程中会在`/usr/share/applications`目录下写入一个后缀为`desktop`的可执行文本文件。

我们以**vmware-workstation.desktop**为例,看看这个文件里究竟有什么内容。

```
# vmware-workstation.desktop

[Desktop Entry]
Encoding=UTF-8
Name=VMware Workstation
Comment=Run and manage virtual machines
Exec=/usr/bin/vmware %U
Terminal=false
Type=Application
Icon=vmware-workstation
StartupNotify=true
Categories=System;
MimeType=application/x-vmware-vm;application/x-vmware-team;application/x-vmware-enc-vm;x-scheme-handler/vmrc;
```

我们来看看上面的字符是什么意思:
* **[Desktop Entry]**: Desktop Entry 文件标准是由 FreeDesktop.org（http://freedesktop.org/wiki/） 制定的，该标准描述了程序的启动配置信息，类似Windows下的快捷方式。
* **Encoding**: 编码
* **Name**: 应用名称
* **Comment**: 描述信息
* **Exec**: 可执行文件路径
* **Terminal**: 是否启动终端
* **Type**: 类型，应用程序即`Application`即可
* **Icon**: 应用图标的绝对路径
* **Categories**: 程序类别
* ...

对于我们自定义快捷方式一般只需修改
* Categories
* Name
* Comment
* Exec
* Type
* Icon
* Comment[zh_CN] (如果有中文名)

举个栗子:

Moedito是一个开源的Markdown编辑器。

> Built with Electron.

clone代码之后，需要在终端中执行 `electron .` 来启动，我们可以把该命令写在一个shell脚本中(路径需为绝对路径)，然后创建desktop文件，如下:

```
[Desktop Entry]
Categories=Editor
Exec=/home/cofcool/bin/Moeditor/Moeditor.sh
GenericName[zh_CN]=Editor
GenericName=Editor
Icon=/home/cofcool/bin/Moeditor/icons/Moeditor.png
Name[zh_CN]=Moeditor
Name=Moeditor
StartupNotify=true
Terminal=false
Type=Application
```

经过以上处理之后，方便很多吧。
