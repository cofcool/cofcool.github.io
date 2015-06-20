---
layout: post
category : Tech
title : Ubuntu美化技巧
tags : [菜鸟的Ubuntu学习之旅]
---
{% include JB/setup %}


在加入Linux世界之前，一定经常听人说Linux只适合服务器，或是Linux界面不美观之类的话。不过，现在早已不是计算机史前时代了，各个操作系统厂商很注重提高Linux系统UI的使用体验。虽然厂商已经做的很好了，不过Linux是自由的，因此我们可以自由定制UI界面，可以模仿OS X或是chrome OS等OS的主题，也可以自由的更改图标或是窗口主题等。

更改系统主题：

**方法1**  直接安装主题，以“Paper”主题为例：

  1. 安装主题
  
         sudo add-apt-repository ppa:snwh/pulp
         sudo apt-get update
         sudo apt-get install paper-gtk-theme paper-icon-theme
		
  2. 启用
  
   使用系统调整工具更改系统主题。在这里，使用Ubuntu Tweak进行调整。（系统调整工具看我上一篇文章：[Ubuntu系统调整工具](http://www.cofcool.net/tech/2015/06/25/ubuntu-system-configure-tools/)）
    
   具体过程截图：
    
   ![image]({{ site.url }}/public/upload/images/shot0042.png)
    
   ![image]({{ site.url }}/public/upload/images/shot0043.png)
    
   ![image]({{ site.url }}/public/upload/images/shot0044.png)
    
   ![image]({{ site.url }}/public/upload/images/shot0045.png)
    
   ![image]({{ site.url }}/public/upload/images/shot0046.png)

**方法2** 已有主题文件，复制到指定目录：

  1. 主题文件复制
  
   把下载到的主题文件复制到用户**home**目录下的**.themes**文件夹中，如果没有创建即可。要是还有icon主题，则把它复制到到**.icons**文件夹中。
    
  2. 启用
  
   方法同上。
    
---

通过以上方法，就可以根据自己的喜好来设置系统主题了。