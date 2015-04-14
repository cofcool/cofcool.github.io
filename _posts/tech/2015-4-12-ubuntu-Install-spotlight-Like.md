---
layout: post
category : Tech
title : 在ubuntu上实现spotlight
tags : [菜鸟的Ubuntu学习之旅,软件]
---
{% include JB/setup %}



Mac OS X中有一个很实用搜索的功能Spotlight，用户可以容易的通过它来完成一些轻量级文件软件搜索任务。虽然Ubuntu上也有实现类似的功能，但是得通过打开“dash”来搜索，略有不便。不过，现在通过“indicator-synapse”可以实现类似Spotlight的体验，虽然功能有限，但是也足以使用了。
![indicator-synapse]({{ site.url }}/public/upload/images/shot0025.png)

![indicator-synapse]({{ site.url }}/public/upload/images/shot0026.png)

下面我们来安装indicator-synapse：

	sudo add-apt-repository ppa:noobslab/apps
	sudo apt-get update
	sudo apt-get install indicator-synapse
	
重启之后，就可以使用了。