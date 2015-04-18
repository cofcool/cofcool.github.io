---
layout: post
category : Tech
title : Ubuntu更换grub背景图片
tags : [菜鸟的Ubuntu学习之旅]
---
{% include JB/setup %}

我相信大家一定受够了PC开机时很蠢，古老……的启动画面，不管什么品牌，型号，它们的启动画面似乎永远都是“黑底白字”。都什么年代了，还在使用N年前的风格，没有一家PC厂商想要改变。不过，在自由的开源世界一切都有可能。那就是Grub，这是一个极其强大的系统引导软件。你可以随意更改配置，只要你愿意。好了废话不多说了，无图无真相，图来了：
![image]({{ site.url }}/public/upload/images/shot0027.jpg)


**现在我们来自定义grub背景图片**

1.复制图片
	
	sudo cp image /boot/grub

2.update Grub
	
	sudo update-grub
	
***so-o easy！***