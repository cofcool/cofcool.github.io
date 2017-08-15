---
layout: post
category : Tech
title : Pure-FTPd配置
tags : [ftp, ops]
---
{% include JB/setup %}

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 安装](#1-安装)
* [2. 配置](#2-配置)

<!-- /code_chunk_output -->


## 1. 安装

```
sudo apt-get install pure-ftpd-mysql
```

## 2. 配置

    sudo groupadd -g 2001 ftpgroup
    sudo useradd -u 2001 -s /bin/false -d /dev/null -c "Pure-FTPd User" -g ftpgroup ftpuser
    sudo sh -c "echo 'yes' > /etc/pure-ftpd/conf/ChrootEveryone"
    sudo sh -c "echo 'No' > /etc/pure-ftpd/conf/CreateHomeDir"

    mysql -u root -p

    CREATE DATABASE ftpusers;
    GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP ON ftpusers.* TO 'ftpadmin'@'localhost' IDENTIFIED BY 'ftpadminPassword';
    FLUSH PRIVILEGES;
    USE ftpusers;
    CREATE TABLE IF NOT EXISTS `users` (
      `User` varchar(16) NOT NULL DEFAULT '',
      `Password` varchar(32) NOT NULL DEFAULT '',
      `Uid` int(11) NOT NULL,
      `Gid` int(11) NOT NULL,
      `Dir` varchar(128) NOT NULL DEFAULT '',
      `QuotaFiles` int(10) NOT NULL DEFAULT '500',
      `QuotaSize` int(10) NOT NULL DEFAULT '30',
      `ULBandwidth` int(10) NOT NULL DEFAULT '80',
      `DLBandwidth` int(10) NOT NULL DEFAULT '80',
      `Ipaddress` varchar(15) NOT NULL DEFAULT '*',
      `Comment` tinytext,
      `Status` enum('0','1') NOT NULL DEFAULT '1',
      `ULRatio` smallint(5) NOT NULL DEFAULT '1',
      `DLRatio` smallint(5) NOT NULL DEFAULT '1',
      PRIMARY KEY (`User`)
    ) ENGINE=MyISAM DEFAULT CHARSET=utf8;

    sudo cp /etc/pure-ftpd/db/mysql.conf /etc/pure-ftpd/db/mysql.conf.bk
    sudo vim /etc/pure-ftpd/db/mysql.conf

    /etc/pure-ftpd/db/mysql.conf

        MYSQLServer     127.0.0.1

        MYSQLSocket      /var/run/mysqld/mysqld.sock

        MYSQLUser       ftpadmin

        MYSQLPassword   ftpadminPassword

        MYSQLDatabase   ftpusers

        MYSQLCrypt      md5

        MYSQLGetPW      SELECT Password FROM users WHERE User='\L' AND Status="1" AND (Ipaddress = "*" OR Ipaddress LIKE "\R")

        MYSQLGetUID     SELECT Uid FROM users WHERE User='\L' AND Status="1" AND (Ipaddress = "*" OR Ipaddress LIKE "\R")

        MYSQLGetGID     SELECT Gid FROM users WHERE User='\L' AND Status="1" AND (Ipaddress = "*" OR Ipaddress LIKE "\R")

        MYSQLGetDir     SELECT Dir FROM users WHERE User='\L' AND Status="1" AND (Ipaddress = "*" OR Ipaddress LIKE "\R")

        MySQLGetQTAFS  SELECT QuotaFiles FROM users WHERE User='\L' AND Status="1" AND (Ipaddress = "*" OR Ipaddress LIKE "\R")

        MySQLGetQTASZ  SELECT QuotaSize FROM users WHERE User='\L' AND Status="1" AND (Ipaddress = "*" OR Ipaddress LIKE "\R")

        MySQLGetRatioUL SELECT ULRatio FROM users WHERE User='\L' AND Status="1" AND (Ipaddress = "*" OR Ipaddress LIKE "\R")
        MySQLGetRatioDL SELECT DLRatio FROM users WHERE User='\L' AND Status="1" AND (Ipaddress = "*" OR Ipaddress LIKE "\R")

        MySQLGetBandwidthUL SELECT ULBandwidth FROM users WHERE User='\L' AND Status="1" AND (Ipaddress = "*" OR Ipaddress LIKE "\R")
        MySQLGetBandwidthDL SELECT DLBandwidth FROM users WHERE User='\L' AND Status="1" AND (Ipaddress = "*" OR Ipaddress LIKE "\R")

    sudo chmod g=o= /etc/pure-ftpd/db/mysql.conf
    sudo service pure-ftpd-mysql restart

需要注意的是，用户必须拥有对应的ftp目录的权限，而且用户的GID和UID必须和配置的相符。创建一个目录，如下：

    sudo mkdir ftp-dir
    sudo chown ftpuser:ftpgroup -R  ftp-dir

如上配置，凡是新建的ftp用户可拥有对该目录的权限。新建用户的使用，可以在数据库中插入用户数据即可，用户密码数据插入时需使用`MD5`加密，如下所示：

    INSERT INTO `users`
        (`User`, `Password`, `Uid`, `Gid`, `Dir`, `QuotaFiles`, `QuotaSize`, `ULBandwidth`, `DLBandwidth`, `Ipaddress`, `Comment`, `Status`, `ULRatio`, `DLRatio`)
        VALUES
        ('ftp', md5('cofcool.net'), 2001, 2001, '/home/ftp-dir/', 500, 30, 80, 80, '*', 'ftp home dir', '1', 1, 1);
