---
layout: post
category : Tech
title : 快速配置 Let’s Encrypt 证书
tags : [ops]
excerpt: 最近几乎所有网站都支持"HTTPS"，但是个人小站购买证书价钱较贵，不过幸好 Let’s Encrypt 提供了免费的证书，并且容易使用配置。
---
{% include JB/setup %}

参考网站 :

* [Let’s Encrypt](https://letsencrypt.org/)
* [certbot](https://certbot.eff.org)

最近几乎所有网站都支持"HTTPS"，但是个人小站购买证书价钱较贵，不过幸好 [Let’s Encrypt](https://letsencrypt.org/) 提供了免费的证书，并且容易使用配置。

本文以 `certbot v1.0.0` 为例，环境为`Nginx on CentOS/RHEL 7`。

#### 安装：

```
# CentOS 添加 EPEL 源
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

# 安装依赖工具
yum -y install yum-utils
yum-config-manager --enable rhui-REGION-rhel-server-extras rhui-REGION-rhel-server-optional

# 安装 certbot
yum install certbot python2-certbot-nginx
```

#### 配置

执行如下命令，按照提示配置即可。如成功会提示`ongratulations! You have successfully enabled https://...`

```
certbot --nginx
```

证书的有效期只有三个月，可配置自动续期，执行如下命令自动续期即可。

```
echo "0 0,12 * * * root python -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```

#### 常见问题

##### 1. urllib3 错误

> ImportError: cannot import name UnrewindableBodyErro

```
pip install urllib3==1.21.1
```

##### 2. pyOpenSSL 版本问题

> ImportError: 'pyOpenSSL' module missing required functionality. Try upgrading to v0.14 or newer.


系统内置的 pyOpenSSL(0.13) 版本太旧，，安装新版即可([pyOpenSSL on pkgs.org](https://pkgs.org/download/pyOpenSSL))

```
yum rmeove pyOpenSSL
wget http://ftp.ksu.edu.tw/FTP/CentOS/7/cloud/x86_64/openstack-ocata/common/pyOpenSSL-0.15.1-1.el7.noarch.rpm
rpm -Uvh python2-pyOpenSSL-16.2.0-4.el7.noarch.rpm
```


---

几年前的草稿

```sh
$ wget https://dl.eff.org/certbot-auto
$ sudo ./path/to/certbot-auto --apache

Automating renewal

Certbot can be configured to renew your certificates automatically before they expire. Since Let's Encrypt certificates last for 90 days, it's highly advisable to take advantage of this feature. You can test automatic renewal for your certificates by running this command:

$ sudo ./path/to/certbot-auto renew --dry-run

If that appears to be working correctly, you can arrange for automatic renewal by adding a cron job or systemd timer which runs the following:

$ ./path/to/certbot-auto renew

Note:

if you're setting up a cron or systemd job, we recommend running it twice per day (it won't do anything until your certificates are due for renewal or revoked, but running it regularly would give your site a chance of staying online in case a Let's Encrypt-initiated revocation happened for some reason). Please select a random minute within the hour for your renewal tasks.

An example cron job might look like this, which will run at noon and midnight every day:

# 0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && ./path/to/certbot-auto renew```


$ ./certbot-auto --apache -d example.com

```
