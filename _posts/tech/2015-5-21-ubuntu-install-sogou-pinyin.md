---
layout: post
category : Tech
title : Ubuntu安装搜狗输入法
tags : [菜鸟的Ubuntu学习之旅]
---
{% include JB/setup %}


Ubuntu自带的IBus输入法对中文不太友好，智能联想功能比较弱，很多词组不能智能匹配。因此，需要一款更加强大的输入法，中文输入法中**搜狗输入法 for Linux**还是挺不错的，具体的介绍可以去搜狗输入法官网了解，[链接](http://pinyin.sogou.com/linux/)。

##安装：

系统环境 ：Ubuntu 15.04

1. 下载deb安装包：


2. 安装：

        双击安装包，使用“软件管理中心”打开，安装即可
        
3. 安装必须组件

        sudo apt-get install fcitx-table
        
4. 添加和配置搜狗输入法

	1） 打开系统设置，点击“语言支持”，把“Keyboard input method system”改为“fcitx”
	
	![image]({{ site.url }}/public/upload/images/shot0033.png)

	2） 重启之后，在右上角菜单区域会显示“fcitx”输入法图标，点击可以发现并没有“搜狗输入法”。点击“配置”来添加搜狗输入法。
	
	![image]({{ site.url }}/public/upload/images/shot0034.png)
	
	3） 点击“+”添加，输入“sogou”搜索，选中点击“OK”即可。
	
	![image]({{ site.url }}/public/upload/images/shot0035.png)	
	
	![image]({{ site.url }}/public/upload/images/shot0036.png)
        
    4） 等待片刻，重新点击输入法图标即可发现搜狗输入法。
    
    ![image]({{ site.url }}/public/upload/images/shot0037.png)	
    
    ![image]({{ site.url }}/public/upload/images/shot0038.png)
    
安装结束。