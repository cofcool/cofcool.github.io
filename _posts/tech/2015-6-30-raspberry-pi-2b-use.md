---
layout: post
category : Tech
title : 树莓派的简单使用
tags : [树莓派]
---
{% include JB/setup %}


前一段时间，入手了树莓派2B。之前经常在新闻上看到**树莓派**的相关报道，很好奇“树莓派究竟是一个什么东西”，从媒体报道来看，只知道是个开发板，搭载ARM机构的CPU，可以完成一些简单的任务。

最近，突然发现仅仅经常为了看个视频就开电脑，不看就把电脑休眠或是关机了，似乎没有发挥电脑的价值，而且这样比较麻烦。而且，macbook接口比较少，且不能读写NTFS格式的硬盘，每次想要拷贝数据需要来回在两个电脑之间切换，很是麻烦、费劲。于是在想“有没有什么方案可以解决这个问题”，既可以分享文件也可以用作多媒体中心。

正好自己安装了Ubuntu，于是使用samba和kodi来解决文件共享和视频观看的问题。但是这样还是得常开一台电脑，还会有诸多不便。

突然想起还有“树莓派”这样的开发板，似乎可以安装linux系统，于是在网上查了查，发现树莓派官网提供了Ubuntu core，Ubuntu Mate，基于Debian的
Raspbian以及OSMC等等linux系统和多媒体管理中心。而且，树莓派只需要内存卡就可以，可以准备多个内存卡，刻录多个系统。看样子，树莓派应该可以做linux服务器来提供samba或NFS服务，也可以当多媒体中心来使用了。于是在万能的淘宝上买了一个raspberry pi 2B，因为树莓派只支持输出HDMI，我只有一台VGA接口的显示器，于是又买了一个HDMI转VGA线。

官方提供的部分系统：

![image]({{ site.url }}/public/upload/images/0053.png)

等待的时间总是漫长的，于是拿出以前用过的内存卡，使用烧录软件把系统镜像烧录到内存卡上，做了OSMC和Raspbian两个系统内存卡。三天之后，终于到了，迫不及待的把内存卡插好，使用手机充电器作为电源插好。

先使用的系统是Raspbian，但是出了一个重要的问题，插上显示器没没反应。不知道是转接线还是系统的问题，试了多次还是没办法。只好使用SSH远程连接试试，但是不知道IP地址，无法连接啊，崩溃！！！

因为我在一个局域网里，有很多电脑都连接路由器，根本不知道哪一个是PI。只好打开路由器，查看IP地址映射表，一个个的尝试连接。多次尝试之后，终于找到了PI，使用SSH连接上了。因为Raspbian是基于Debian的，使用apt-get包管理系统，所以系统很快上手了，整体感觉还不错，用起来也不卡顿，系统安装的服务和软件比较少，CPU和内存占用率都比较低。但是Raspbian安装samba时，没有smbpasswd命令，创建不了samba用户，并且发现一些命令没有，也不知道怎么安装和设置。于是重新刻录了Ubuntu Mate树莓派版本，这次倒是很正常的连接了显示器，系统也正常安装设置好了。

Ubuntu Mate树莓派版本和Ubuntu Mate桌面版使用基本一样，很快安装了samba，设置好用户以及路径等选项之后，重启samba服务，就可以使用文件分享了，其它电脑也可以正常挂载分享的磁盘了。不过，我只想使用samba，没有必要开启图形界面，于是关闭了图形界面，开机之后，使用SSH登陆，并挂载磁盘，就可以了。需要注意的是，系统处于文本界面时不会自动挂载磁盘，需要手动挂载，如果想要自动挂载，可以在/etc/fstab文件中添加磁盘信息就可以了。（我之前写的samba简单配置的说明，[查看](http://www.cofcool.net/tech/2015/05/29/ubuntu-install-samba-and-configure/)）

**需要注意**的是树莓派的USB供电不足，直接使用移动硬盘的话会供电不足而无法正常使用，因此使用移动硬盘的话，需要外界电源来供电，可以使用带电源的USB HUB也可以使用带电源的3.5英寸移动硬盘。HDMI-VGA转接线也是如此，需要使用带电源的转接线。

树莓派的配置过程大概就是这样了，大家如果有兴趣的话可以查查树莓派相关知识，自己尝试一下。

图片：

![image]({{ site.url }}/public/upload/images/0056.jpg)

![image]({{ site.url }}/public/upload/images/0057.jpg)

![image]({{ site.url }}/public/upload/images/0058.jpg)

![image]({{ site.url }}/public/upload/images/0054.jpg)

![image]({{ site.url }}/public/upload/images/0055.jpg)

![image]({{ site.url }}/public/upload/images/0059.jpg)

![image]({{ site.url }}/public/upload/images/0060.jpg)