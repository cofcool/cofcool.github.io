---
layout: post
category : Tech
title: 安装ubuntu之后,需要做什么
tagline: "Ubuntu相关"
tags : [菜鸟的Ubuntu学习之旅, 技巧]
---
{% include JB/setup %}




当我们怀着对ubuntu的渴望与好奇而安装她之后,突然发现ubuntu根本不是想像中的那么美好。她像一个亡国的公主, 空有美貌而无一用处,连最简单的事她都做的一塌糊涂。这时,千万不要灰心气馁,要坚持下去,就像对待一个不懂事的小女孩一样,要有足够的耐心。好了,废话不说了,我们进入正题。
### 1. 更换软件源:

当进入桌面之后,首先打开"system settings",如下所示:

![image]({{ site.url }}/public/upload/images/shot0021.jpg)

然后点击"software&update",出现如下窗口,选择"Download from":
![image]({{ site.url }}/public/upload/images/shot0023.jpg)

根据自己的网络情况来选择软件源,或者之直接点击"Select best server"。网通的网可以选择souhu的源。

![image]({{ site.url }}/public/upload/images/shot0022.jpg)

选择完成后,根据提示点击"reload",
或自己打开终端,输入如下命令:

	sudo apt-get update

### 2. 安装Nvidia显卡管理软件:

由于linux kernel的电池管理软件似乎不太有用,并且显卡驱动由于是通用驱动,所以电脑发热很厉害,特别费电。因此我们需要一个强大的电池管理软件.我推荐大家使用bumblebee。大家可以到以下[地址](http://bumblebee-project.org/)查看,根据自己的系统选择安装.bumblebee官网。安装之后,独立显卡会关闭,使用如下命令查看

	lspci |grep VGA

如果显示如下(显示结果因电脑不同可能会有差别)

	￼01. ￼ VGA compatible controller: NVIDIA Corporation Device 0de9 (rev ff)

出现"ff",这说明成功关闭. 可参考http://forum.ubuntu.org.cn/viewtopic.php?f=126&t=374321

### 3. 关闭错误提示:

由于linux桌面版目前还不是特别完善,因此会有各种各样的问题.经常在使用的时候弹出"错误提示",搞的人很烦.因为
linux是自由的,所以我们可以随心所欲的更改系统文件。so,我们可以关闭"错误提示".命令如下

	sudo vi /etc/default/apport

然后把"enabled=1"改为"enabled=0",就可以关闭错误提示了。

### 4. 更改电脑屏幕亮度:

acer电脑的某些版本安装ubuntu之后来亮度不可以调节,因此我们需要修改系统文件来调节亮度。

	echo 320 > /sys/class/backlight/intel_backlight/brightness

注意:root用户才可以执行上述命令."320"为亮度大小,可以根据自己的要求来更改。要想开机自动更改亮度,使用如下命令

	echo 'echo 320 > /sys/class/backlight/intel_backlight/brightness' > /etc/rc.local
另外,需要找到/etc/default/grub文件,把

	GRUB_CMDLINE_LINUX=""

改为

	GRUB_CMDLINE_LINUX="acpi_backlight=vendor"

再更新grub

	sudo update-grub

### 5. gedit打开中文时乱码

我们安装ubuntu之后,因为ubuntu为英文国家所开发的系统,所以会缺少我们中文的字符编码。所以我们需要添加中文编码。

	sudo gedit /var/lib/locales/supported.d/local

打开之后在文本中添加字符编码,如下所示:zh_CN.GBK GBK来添加GBK编码。添加保存之后,执行

	sudo dpkg-reconfigure locales

对于gedit3,还可以

	gsettings set org.gnome.gedit.preferenceected "['GB18030', 'UTF-8', 'GB2312', 'GBK', 'BIG5', 'CURRENT', 'UTF-16']"

还有很多小技巧，在之后的文章会继续介绍。
