---
layout: post
category : Tech
title : Ubuntu读写exFAT文件系统的U盘
tags : [菜鸟的Ubuntu学习之旅]
---
{% include JB/setup %}


我们经常会下载一些高清或是蓝光的电影，这些文件体积一般都比较大，如果使用U盘的话，会因为文件过大无法存储。不同的存储设备可能会使用不同的文件系统，一般PC使用的是NTFS文件系统，Mac使用HFS+，U盘一般使用的是FAT文件系统。FAT32无法存储大于4GB的单个文件，所以要想存储大于4GB的文件需要使用NTFS或是exFAT等，不过在Mac系统中NTFS只读不可写。因此要想在Mac使用U盘存储大于4GB的文件需要使用把U盘格式化为exFAT文件系统。但是在Ubuntu系统中，因为文件系统专利的问题不可以读写，所以采用第三方方案。

安装exFAT文件系统内核补丁：
    
ubuntu 14.04

    sudo apt-get install exfat-utils
    
ubuntu 12.04 ~ 13.10

    sudo add-apt-repository ppa:relan/exfat
    sudo apt-get update
    sudo apt-get install exfat-utils fuse-exfat
    
重启之后就可以读写exFAT文件系统的U盘了。
