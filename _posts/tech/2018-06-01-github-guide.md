---
layout: post
category : Tech
title : Github简单使用
tags : [git]
---
{% include JB/setup %}

Github在开发中的作用越来越重要，会频繁的使用开源软件和开源代码。那么，如何自己创建一个项目。

目录:
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=2 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 创建项目](#1-创建项目)
* [2. 克隆](#2-克隆)
* [3. 项目提交](#3-项目提交)

<!-- /code_chunk_output -->

## 1. 创建项目

创建项目

![github-01](http://cofcool.net/imgs/github-01.png)

填写项目名，项目描述，项目版权许可证等，如果是`公共项目`，选择`Public`即可（免费），`私有项目`选择`Private`（需付费），如果有README文件可选`初始化项目并创建README`。
![github-02](http://cofcool.net/imgs/github-02.png)

![github-03](http://cofcool.net/imgs/github-03.png)

叮，项目创建成功。

## 2. 克隆

拷贝项目地址：

![github-04](http://cofcool.net/imgs/github-04.png)

命令：
```sh
git clone https://github.com/cofcool/github-demo.git
cd github-demo
```

## 3. 项目提交

修改

```sh
echo "GitHub 简单使用。" >>  README.md
```

提交

```sh
git add README.md
git commit -m "update README.md"

git status
# 输出状态
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

推送到GitHub，如提示需要`username`,`password`，正确输入即可。

```sh
git push

# 输出状态
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 313 bytes | 313.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/cofcool/github-demo.git
   e62cbe7..d6f680e  master -> master
```

我们来看看最后成果。
![github-04](http://cofcool.net/imgs/github-05.png)

注意⚠️: *Github官方推荐使用URL为`HTTP`的方式来授权验证，只需要用户名和密码即可，如果使用`SSH`的方式，需要验证`SSH KEY`来做授权。详情可查看[Which remote URL should I use?](https://help.github.com/articles/which-remote-url-should-i-use/)。*

简单几步即可在GitHub上创建一个项目，快来为开源软件做贡献吧。
