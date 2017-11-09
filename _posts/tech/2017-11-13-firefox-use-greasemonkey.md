---
layout: post
category : Tech
title : 油猴脚本安装
tags : [firefox]
---
{% include JB/setup %}

环境: 已安装脚本管理器的浏览器



大名鼎鼎的油猴，想必大家都有耳闻，今天我们来看看如何安装自己的脚本文件。

在[Greasemonkey wiki](https://wiki.greasespot.net/User_Script_Hosting)页面我们可以看到，有四种方式安装脚本：

* Greasy Fork
* OpenUserJS.org
* Gist
* The Whole Internet

通过前两种方法可以安装已发布的脚本，对于我们自己写的脚本可通过后两种方法安装。

把自己写的脚本文件发布在web服务器上，并以`.user.js`为后缀名，访问该链接即可安装。没有自己的web服务器，可通过GithubGist服务来发布自己的脚本代码。如果使用的浏览器是Firefox，不能通过GithubGist来安装。

> Warning: A [Firefox bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1411641) prevents user script installation from some sites, including Github and Gist, from working as intended.

上述问题的解决方案：

在本地启动一个web服务器：

```shell
python -m SimpleHTTPServer 8000
```

然后访问该文件即可。

以移除掘金展开全文按钮为例:

脚本文件:

<script src="https://gist.github.com/gofcool/87d3b41ef32021c575df1ca206d303d2.js"></script>

点击该页面的[Raw](https://gist.githubusercontent.com/gofcool/87d3b41ef32021c575df1ca206d303d2/raw/4d9b1b1e40821d76cce55674c50bee6d6f38c09c/remove-juejin.user.js)，即可安装。
