---
layout: post
category : Tech
title : GitLab Jenkins 持续集成
tags : [git,ops]
---
{% include JB/setup %}

目录
<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 安装](#1-安装)
* [2. 配置](#2-配置)
* [3. 添加任务](#3-添加任务)
* [4. 常见问题](#4-常见问题)
	* [1. 忘记密码](#1-忘记密码)
* [参考](#参考)

<!-- /code_chunk_output -->


环境:
* Ubuntu 16.04 LTS
* OpenJDK 8
* GitLab 9

## 1. 安装

```shell
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -

sudo vim /etc/apt/sources.list
deb https://pkg.jenkins.io/debian-stable binary/

sudo apt update && sudo apt install jenkins
```

## 2. 配置

```shell
# 修改端口，默认为8080， 修改为 11111
sudo vim /etc/default/jenkins

# port for HTTP connector (default 8080; disable with -1)
HTTP_PORT=11111

# 重启服务
sudo service jenkins restart
```

如果启动成功，可访问`http://%{YOUR_IP_ADDRESS}:11111/`来进行初始化。

1. 输入`/var/lib/jenkins/secrets/initialAdminPassword`文件中的密码解锁
2. 开始初始化配置，如下图所示
  ![zdtefkofcy6nu3di]({{ site.url }}/public/upload/images/zdtefkofcy6nu3di.png)
  ![ev9w7nrmspbn8kt9]({{ site.url }}/public/upload/images/ev9w7nrmspbn8kt9.png)
  ![coewnqfz9ja46lxr]({{ site.url }}/public/upload/images/coewnqfz9ja46lxr.png)
3. 按照要求配置完成之后，即可访问
  ![5adt4ll1jfbf0f6r]({{ site.url }}/public/upload/images/5adt4ll1jfbf0f6r.png)
4. 安装Gitlab插件, GitLab Plugin 和 Gitlab Hook Plugin
  ![bnnf1fdzte1jdcxr]({{ site.url }}/public/upload/images/bnnf1fdzte1jdcxr.png)

## 3. 添加任务

根据Jenkins提供的向导来创建任务，大概过程如下:

![kbqpt4ikug2jra4i]({{ site.url }}/public/upload/images/kbqpt4ikug2jra4i.png)

![pzkv1mjbd4fmkj4i]({{ site.url }}/public/upload/images/pzkv1mjbd4fmkj4i.png)

注意：Gitlab如果设置为私有项目的话，需要SSH key验证，

![gz2moaiq0e0qw7b9]({{ site.url }}/public/upload/images/gz2moaiq0e0qw7b9.png)

![dv1k7qrb9p9tqpvi]({{ site.url }}/public/upload/images/dv1k7qrb9p9tqpvi.png)

项目创建完成之后，配置Gitlab的Web Hooks：

[Jenkins Gitlab Hook Plugin](https://github.com/elvanja/jenkins-gitlab-hook-plugin)提供的Web Hooks方式:
* Build now hook `http://your-jenkins-server/gitlab/build_now`，如果未触发，可改为`http://your-jenkins-server/gitlab/build_now/your-project-name`
* Notify commit hook `http://your-jenkins-server/git/notifyCommit?url=<URL of the Git repository for the Gitlab project>`

可在Gitlab的项目`设置 > 集成`下设置，如下所示：

![30u6sbe5diyvte29]({{ site.url }}/public/upload/images/30u6sbe5diyvte29.png)

![3rthak1175koi529]({{ site.url }}/public/upload/images/3rthak1175koi529.png)

添加完成之后，可点击`Test`测试是否成功。

![79z7v6zdlg1u4n29]({{ site.url }}/public/upload/images/79z7v6zdlg1u4n29.png)

## 4. 常见问题

### 1. 忘记密码

用户的配置文件存储位置为`/var/lib/jenkins/users`，可通过修改用户目录下的`config.xml`来重置密码。

```
# 123456 对应的密码加密后的值
#jbcrypt:$2a$10$NqPv3NpgxkpQi/ffEsEkhuMZYpbKc5cVVrP60cD6MX5IujYkLlOGm
```

## 参考

* [Jenkins Gitlab持续集成打包平台搭建](http://skyseraph.com/2016/07/18/Tools/Jenkins%20Gitlab%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E6%89%93%E5%8C%85%E5%B9%B3%E5%8F%B0%E6%90%AD%E5%BB%BA/)
