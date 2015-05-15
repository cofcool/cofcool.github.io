---
layout: post
category : Tech
title : Ubuntu安装NVIDIA显卡驱动
tags : [菜鸟的Ubuntu学习之旅]
---
{% include JB/setup %}


>
作为一个linux用户，双显卡问题永远是心中的痛！AMD和NVIDIA似乎从来不在乎linux平台的用户需求，他们提供的驱动就像一台老式蒸汽机一样，让电脑风扇狂转，发热巨大。**显卡驱动导致的费电和发热的问题**，让许多喜爱linux的用户望而却步。不过随着linux的发展和硬件厂商的发力，产生了许多显卡解决方案，常用的有Bumblebee和Nvidia prime项目。Bumblebee是社区项目，虽然可以解决一些问题，但是兼容性和功能略有不足。而Nvidia Prime由NVIDIA和linux内核社区支持，保证了兼容性和功能的持续支持。下面，我们来看看Nvidia Prime项目(*显卡驱动只能安装其中一个*)。

实验环境：   
系统: Ubuntu 14.04 LTS   
显卡: NVIDIA GT 630M，Intel HD4000  

**1.关闭自带的开源显卡驱动**   

打开终端，执行：

	sudo gedit /etc/modprobe.d/blacklist
	
在文件的末端添加：

	blacklist nouveau
	
**2.安装NVIDIA显卡驱动**

把系统自带驱动加入黑名单之后，开始安装显卡驱动，如下：

	sudo apt-get install nvidia-319 nvidia-settings-319
	
安装显卡驱动结束后，安装显卡切换程序：	

	sudo apt-get install nvidia-prime
	
安装重启之后，显卡驱动就生效了。我们可以通过**prime-select**命令来切换Intel显卡和NVIDIA显卡。

**3.安装双显卡切换指示器**

安装安装双显卡切换指示器之后，可以在系统右上角系统菜单区点击指示器来切换显卡。

    sudo add-apt-repository ppa:nilarimogard/webupd8
    sudo apt-get update
    sudo apt-get install prime-indicator


![image]({{ site.url }}/public/upload/images/shot0030.jpg)
