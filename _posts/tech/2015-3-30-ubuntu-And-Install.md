---
layout: post
category : Tech
title : Ubuntu简介以及安装
tagline: "Ubuntu的安装"
tags : [菜鸟的Ubuntu学习之旅, Linux]
---
{% include JB/setup %}



### 1. Ubuntu的简单介绍
Ubuntu 是一个以桌面应用为主的linux操作系统,其名称来自非洲南部祖鲁语或豪萨语的“ubuntu”一词,意思是 “人性”、“我的存在是因为大家的存在”,是非洲传统的一种价值观,类似华人社会的“仁爱”思想。Ubuntu基于 Debian发行版和GNOME桌面环境,与Debian的不同在于它每6个月会发布一个新版本。Ubuntu的目标在于为一般 用户提供一个最新的、同时又相当稳定的主要由自由软件构建而成的操作系统。Ubuntu具有庞大的社区力量,用 户可以方便地从社区获得帮助。
在我看来,linux的众多桌面版本中,ubuntu的最容易上手的一个版本。它不仅拥有华丽的桌面,还拥有简洁的 文件管理系统,同时拥有众多开源社区的开源软件,足够满足个人的需要。另外,CCN与中国Ubuntu社区携手,开 发了UbuntuKylin ,此次版本包括了为中国市场精心打造的特制体验,给予中国OEM一个给中国电脑用户提供本土 化体验的机会,可通过此网址免费下载.该版本加入了中国特有的元素,很适合中国人使用。
![ubuntu_images]({{ site.url }}/public/upload/images/shot0016.gif)
![ubuntu_images]({{ site.url }}/public/upload/images/shot0017.gif)
![ubuntu_images]({{ site.url }}/public/upload/images/shot0018.gif)
### 2. 安装
首先我们要准备ubuntu的安装镜像,可以直接从官网点击下载,也可以点击以下地址下载:ubuntu的镜像下载。
下载好镜像文件之后,准备一个空白U盘或者VCD光盘,来刻录下载的iso文件。 因为条件限制,无法获取真机的图片,所以我会用VirtualBox来做演示。 好了,废话不多说了,我们马上开始一种全新的操作系统体验吧! 因为安装过程比较繁琐,所以我会使用较多的图片来展示安装过程。


 1. 开始安装:使用VirualBox开始安装ubuntu。![image]({{ site.url }}/public/upload/images/shot0001.gif)
 2. 开始安装,可以选择“试用”或直接开始安装。在这儿,我选择试用。![image]({{ site.url }}/public/upload/images/shot0002.gif)
 3. 进入桌面后,可以先体验一下ubuntu,如果觉得不错的话,可以双击桌面上的“安装ubuntu......”来安装。之 后,选择语言,在这儿我选择中文(简体)。![image]({{ site.url }}/public/upload/images/shot0003.gif)
 4. 安装过程中,一定要选择“安装这个第三方软件”,这是第三方解码器,可以播放mp4等文件。另外不要选中 “安装中下载更新”,否则会耗费很多时间。![image]({{ site.url }}/public/upload/images/shot0005.gif)
 5. 选择安装方式,如果想要在整块磁盘上安装ubuntu的话可以选择“清除整个磁盘并安装ubuntu”。不过,我想各位 朋友如果只是想要体验ubuntu的话,没必要清除整个磁盘来安装。大家应该是想要安装“双系统”,所以我们选择 “其他选择”,然后点击“继续”。![image]({{ site.url }}/public/upload/images/shot0007.gif)
 6. 选择安装位置。在这儿因为是使用虚拟机,磁盘第一次使用,还没有建立分区表,所以先选择“新建分区表”, 如果不是一块全新的硬盘的话,可以略过这一步。![image]({{ site.url }}/public/upload/images/shot0006.gif)
 7. 因为我这儿没有建立好的分区,所以选择“添加”来建立ext4格式的分区,并挂载“/”分区,大小设置为7G,剩下 的1.5G用来设置“swap”交换分区(类似windows下的虚拟内存)。如果你已经拥有建立好的windows“ntfs”分区的 话,可以直接点击“更改”,分区格式选为“ext4”,并选中格式化,大小可以按需要进行设置。如果空间宽裕的话, 可以添加1G-2G大小的“swap”交换分区。(注意:如果在更改或创建分区选中了格式化后,该操作不可逆转,所以应该考虑好之后在点击确定)![image]({{ site.url }}/public/upload/images/shot0010.gif)![image]({{ site.url }}/public/upload/images/shot0012.gif)![image]({{ site.url }}/public/upload/images/shot0004.gif)
 8. 选择好系统安装位置后,可以选择启动引导设备。因为这一步涉及比较多的硬件知识,所以我就不在此解释 了,想要了解的朋友可以使用搜索引擎搜索“grub引导”,“BIOS”,“MBR”等关键字。对此不了解的朋友直接按默认 安装就可以了,然后点击“现在安装”。![image]({{ site.url }}/public/upload/images/shot0002.gif)
 9. 接下来我们选择所在时区,大家选择上海就可以了。![image]({{ site.url }}/public/upload/images/shot0008.gif)
 10. 至于键盘布局的话,根据自己的实际情况选择,我们国内的大部分键盘选择美式布局就可以了。![image]({{ site.url }}/public/upload/images/shot0009.gif)
 11. 接下来,填写账户信息。该步骤可以根据自己的实际情况填写,不过,为了安全起见 ,尽量设置密码,选 择“登录时需要密码”。然后“确认”进行安装。![image]({{ site.url }}/public/upload/images/shot0011.gif)
 12. 经过数十分钟的等待后,安装就可以结束了。重启之后,就可以进入ubuntu的世界了。(注意:某些电脑因 为硬件的问题,可能无法重启,如果出现这种情况后,使用电源键关机,然后再开机。)![image]({{ site.url }}/public/upload/images/shot0013.gif)![image]({{ site.url }}/public/upload/images/shot0014.gif)
