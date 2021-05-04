---
layout: post
category : Tech
title : Ubuntu下Samba的简单配置
tags : [菜鸟的Ubuntu学习之旅]
---
{% include JB/setup %}

>初入linux世界，会有各种各样的问题，特别是使用不同的电脑，当我们想要阅读文件或是其他资料时，不得不使用U盘或是网盘等工具来复制文件，这样不仅麻烦而且费时。因此，需要一种可以提供Windows 与 Unix/类Unix 这两个不同的平台相互分享数据功能的工具。

Samba是在Linux和UNIX系统上实现SMB协议的一个免费软件，由服务器及客户端程序构成。SMB（Server Messages Block，信息服务块）是一种在局域网上共享文件和打印机的一种通信协议，它为局域网内的不同计算机之间提供文件及打印机等资源的共享服务。SMB协议是客户机/服务器型协议，客户机通过该协议可以访问服务器上的共享文件系统、打印机及其他资源。

1. 安装

        sudo apt-get install samba
        
2. 简单的配置

        sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.back
        sudo vim /etc/samba/smb.conf
        
        # 输入以下内容：
        [myshare]
          path = /home/cofcool/share 
          guest ok = no
          valid users = cofcool
          writeable = yes
          
         
        # 测试修改配置是否正确
        testparm
        
        # 创建samba用户
        sudo smbpasswd -a cofcool
        
        # 重启服务
        sudo service smbd restart
        
通过以上操作就可以使用文件分享了。**需要注意的是添加的用户必须对共享文件目录有读写等权限，否则无法连接。**

![image]({{ site.url }}/public/upload/images/shot0039.png)