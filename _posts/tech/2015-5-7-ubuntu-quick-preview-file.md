---
layout: post
category : Tech
title : Ubuntu快速预览文件
tags : [菜鸟的Ubuntu学习之旅]
---
{% include JB/setup %}


当我们想要查看某个文件的内容时，必须通过双击来使用对应的应用程序打开文件，这样不仅麻烦而且比较费时，特别是像视频、文档等体积比较大的文件。有时，不想打开文件，只想看看文件是否含有自己想要的内容。那么，应该怎么办呢？

如果大家使用的是Mac系统的话，可以通过点击“**空格键**”来快速的预览文件内容。不过，现在Ubuntu上可以使用“**空格键**”快速预览文件内容。

方法：

    sudo apt-get install gnome-sushi unoconv
    
通过安装以上所述的软件，即可实现“**空格快速预览文件**”。需要注意的是，并不是所有文件都可以快速预览，另外预览视频需要安装对应格式的解码器，只要安装“ubuntu-restricted-extras”即可。

效果：

Mac ：![image]({{ site.url }}/public/upload/images/shot0028.jpg)

Ubuntu ： ![image]({{ site.url }}/public/upload/images/shot0029.jpg)