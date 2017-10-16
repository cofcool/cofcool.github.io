---
layout: post
category : Tech
title : Ubuntu 卸载旧版内核
tags : [Ubuntu]
---
{% include JB/setup %}

最近在我自己的一台Ubuntu机器上执行`sudo apt update && sudo apt upgrade`时，提示升级失败，内核缺少依赖无法升级。正常情况来说不可能出现升级失败的情况，执行`sudo apt install -f`，依旧失败，但是终端打印出了失败原因，原来是`/boot`分区存储空间已满，导致无法无法安装内核软件包。执行`sudo apt autoremove --purge`来卸载不再需要的旧版内核和安装程序，但是还是执行失败，依旧是内核依赖问题导致。

从以上情况来看，需要手动卸载旧版的内核来释放存储空间。

查看系统分区使用情况:

```Shell
df -h
```

查看当前内核:

```Shell
uname -r
```

列出已安装内核:

```shell
dpkg --get-selections |grep linux-image
```

执行结果如下:

```
linux-image-4.4.0-21-generic			install
linux-image-4.4.0-75-generic			install
linux-image-extra-4.4.0-21-generic		install
linux-image-extra-4.4.0-75-generic		install
```

卸载版本为4.4.0-21的内核(`linux-image-extra-generic`依赖`linux-image-generic`，需要先卸载):

```shell
sudo dpkg --purge linux-image-extra-4.4.0-21-generic linux-image-4.4.0-21-generic
```

移除旧版软件包之后 ，在`/boot`分区有足够的空间之后 ，执行`sudo apt upgrade`重新安装新版软件包即可。