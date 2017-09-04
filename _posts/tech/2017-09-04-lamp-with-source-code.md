---
layout: post
category : Tech
title : 编译源码搭建LAMP和Tomcat
tags : [lamp, tomcat, apache]
---
{% include JB/setup %}

目录：

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. LAMP](#1-lamp)
	* [1. Apache](#1-apache)
		* [1. 下载](#1-下载)
		* [2. apr](#2-apr)
		* [3. apr-util](#3-apr-util)
		* [4. pcre](#4-pcre)
		* [5. httpd](#5-httpd)
	* [2. Mysql(官方源)](#2-mysql官方源)
	* [3. Php](#3-php)
* [2. Tomcat](#2-tomcat)
	* [1. Java](#1-java)
	* [2. Tomcat](#2-tomcat-1)

<!-- /code_chunk_output -->


基础环境:

* Centos 6
* httpd: 2.4
* Mysql: 5.6
* Php: 5.5
* Oracle JDK 8
* Tomcat 8.5


## 1. LAMP

### 1. Apache

#### 1. 下载
```
wget http://mirror.bit.edu.cn/apache/httpd/httpd-2.4.23.tar.gz
tar -xzf httpd-2.4.23.tar.gz
wget http://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-util-1.5.4.tar.gz
wget http://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-1.5.2.tar.gz
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz
tar -xzf apr-1.5.2.tar.gz
tar -xzf apr-util-1.5.4.tar.gz
tar -zxf pcre-8.39.tar.gz
```

#### 2. apr

```
./configure
make && make install
```

#### 3. apr-util

```
./configure  --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
make && make install
```

#### 4. pcre

```
./configure
make && make install
```

#### 5. httpd

```
yum groupinstall 'Development tools'
yum groupinstall 'Server Platform Development'



./configure --prefix=/usr/local/httpd --sysconfdir=/etc/httpd --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-pcre --with-zlib --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util


make && make install

vim /etc/profile.d/httpd24.sh
export PATH=/usr/local/httpd/bin:$PATH
source /etc/profile.d/httpd24.sh

apachectl
```

### 2. Mysql(官方源)

```
vim /etc/yum.repos.d/mysql-community.repo
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=

wget http://repo.mysql.com/RPM-GPG-KEY-mysql
rpm --import RPM-GPG-KEY-mysql

yum makecache
yum install mysql-community-server mysql-community-devel mysql-community-client
```

### 3. Php

```
yum install libxml2 libxml2-devel curl curl-devel libjpeg libjpeg-devel libpng libpng-devel zlib zlib-devel bzip2 bzip2-devel openssl-devel libmcrypt libmcrypt-devel gd php-pdo_mysql freetype freetype-devel

export MYSQL_LIB_DIR=/usr/lib64/mysql

wget http://cn2.php.net/distributions/php-5.5.7.tar.gz

./configure --prefix=/usr/local/php5 --with-config-file-path=/usr/local/php5/etc --with-apxs2=/usr/local/httpd/bin/apxs --with-mysql --with-mysqli --with-freetype-dir=/usr/include/freetype2/freetype/ --with-gd --with-jpeg-dir --with-png-dir --with-iconv --with-zlib --with-zlib-dir --enable-xml --with-mhash --with-pcre-dir=/usr/local/bin/pcre-config --enable-exif --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --enable-fpm --enable-mbstring --enable-ftp --enable-gd-native-ttf --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --with-pear --with-gettext --enable-session --with-mcrypt --with-bz2 --with-curl --enable-dom --with-imap --with-imap-ssl --with-kerberos --with-pdo-mysql

make && make install

cp php.ini-production /usr/local/php5/etc/php.ini
ln -s /usr/local/php5/etc/php.ini /etc/php.ini
```

如果有修改过`mysql.sock`的路径，一方面需要在编译php时指定该目录, 如`configure`时添加如下参数`--with-mysql-sock=${MYSQL_SOCK_PATH}`，并执行如下操作：
```
yum -y install krb5-devel libc-client libc-client-devel
ln -s /usr/lib64/libc-client.so /usr/lib/libc-client.so
```

## 2. Tomcat

### 1. Java

```
wget http://download.oracle.com/otn-pub/java/jdk/8u102-b14/jdk-8u102-linux-x64.tar.gz?AuthParam=1475894298_f560514fc6a875ba3300b5c7c2d755df
tar -zxf jdk-8u102-linux-x64.tar.gz\?AuthParam\=1475894298_f560514fc6a875ba3300b5c7c2d755df
mv jdk1.8.0_102/ /opt/jdk8

vim /etc/rc.local
export JAVA_HOME=/opt/jdk8

vim /etc/profile
export JAVA_HOME=/opt/jdk8
export PATH=${JAVA_HOME}/bin:$PATH
```

### 2. Tomcat

```
wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.5/bin/apache-tomcat-8.5.5.tar.gz
tar -xzf apache-tomcat-8.5.5.tar.gz

mv apache-tomcat-8.5.5 /opt/

vim /etc/rc.d/rc.local
/opt/apache-tomcat-8.5.5/bin/startup.sh start

chmod +x  /etc/rc.d/rc.local

```


⚠️ ***注意***： 下载地址可能失效，仅作参考，可到官网自行下载。
