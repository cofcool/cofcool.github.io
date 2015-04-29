---
layout: post
category : Tech
title : Ubuntu安装Google Chrome浏览器
tags : [菜鸟的Ubuntu学习之旅]
---
{% include JB/setup %}

近年来，Google的chrome发展迅速，很多人已经习惯使用chrome。但是Ubuntu的应用仓库中没有包含chrome的安装包，自己下载安装又有点麻烦以及软件升级不便。那么，应该怎么做了？

我们可以通过添加PPA的方式，把chrome添加Ubuntu的软件源中。具体步骤：

1. 下载Google公钥

	    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
	    
	 需要注意的是，在国内有时因为“众所周知的原因”无法下载Google提供的公钥。我已经下载好了，大家可以到我的百度网盘下载：[下载Google公钥](http://yun.baidu.com/s/1qWJVK9A),下载完成后使用如下命令添加
	 
	    sudo apt-key add linux_signing_key.pub
	    
2. 添加源

		sudo sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
		
3. 更新软件列表

		sudo apt-get update
		
4. 安装

		sudo apt-get install google-chrome-stable
		

通过以上四步就可以使用chrome浏览器了。