# 系统管理神器 osquery



## 1. 简要介绍

[osquery](https://osquery.io/)，这是Facebook为系统管理，运维开发的一款管理工具，适用于OS X/macOS, Windows, and Linux。可以使用SQL直接查询系统环境变量，运行状况，资源占用等。

> osquery is an operating system instrumentation framework for OS X/macOS, Windows, and Linux. 
>
> The tools make low-level operating system analytics and monitoring both performant and intuitive.

<div style="position: relative; width: 100%; height: 100%; background-color: black;"><video preload="none" playsinline="" style="width: 100%; height: 100%; position: absolute; transform: rotate(0deg) scale(1);" poster="https://pbs.twimg.com/tweet_video_thumb/DVNL6TaU0AAZa1P.jpg" src="https://video.twimg.com/tweet_video/DVNL6TaU0AAZa1P.mp4" type="video/mp4"></video></div>

官方示例:

```SQL
osquery> SELECT name, path, pid FROM processes WHERE on_disk = 0;
  name = Drop_Agentpath = /Users/jim/bin/dropagepid = 561

osquery> SELECT * FROM mounts m, disk_encryption dWHERE m.device_alias = d.nameAND m.path = "/"AND d.encrypted = 0;
  device = /dev/disk1
  device_alias = /dev/disk1                          
  path = /              
  type = hfs       
  blocks_size = 4096            
  blocks = 121815040       
  blocks_free = 48994214  
  blocks_available = 48930214            
  inodes = 4294967279       
  inodes_free = 4292826261             
  flags = 75550720              
  name = /dev/disk1              
  uuid = 23446C9A-18F9-4BCF-A088-801E376691FA         
  encrypted = 0              
  type =               
  uid =         
  user_uuid =
```

## 2. 安装

我的实验环境为Fedora 27，所以我们来看看在Linu上如何安装。根据官方文档[Install on Linux](https://osquery.readthedocs.io/en/stable/installation/install-linux/)，可以使用`RPM`包安装，也可使用`YUM源`的方式安装。至于其它系统下的安装可在[官方文档](https://osquery.readthedocs.io/en/stable/)查看。

**RPM**：

```shell
wget https://pkg.osquery.io/rpm/osquery-2.11.2-1.linux.x86_64.rpm
sudo rpm -i osquery-2.11.2-1.linux.x86_64.rpm
```

**YUM**：

```shell
$ curl -L https://pkg.osquery.io/rpm/GPG | sudo tee /etc/pki/rpm-gpg/RPM-GPG-KEY-osquery
$ sudo yum-config-manager --add-repo https://pkg.osquery.io/rpm/osquery-s3-rpm.repo
$ sudo yum-config-manager --enable osquery-s3-rpm
$ sudo yum install osquery

# 也可使用如下方式
$ sudo rpm -ivh https://osquery-packages.s3.amazonaws.com/centos7/noarch/osquery-s3-centos7-repo-1-0.0.noarch.rpm
$ sudo yum install osquery
```

**apt-get**

```shell
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1484120AC4E9F8A1A577AEEE97A80C63C9D8B80B
$ sudo add-apt-repository "deb [arch=amd64] https://osquery-packages.s3.amazonaws.com/xenial xenial main"
$ sudo apt-get update
$ sudo apt-get install osquery
```

## 3. 使用

在终端中输入`osqueryi`即可进入执行环境中。

```shell
[test@localhost ~]$ osqueryi
Using a virtual database. Need help, type '.help'
osquery> .help
Welcome to the osquery shell. Please explore your OS!
You are connected to a transient 'in-memory' virtual database.

.all [TABLE]     Select all from a table
.bail ON|OFF     Stop after hitting an error
.echo ON|OFF     Turn command echo on or off
.exit            Exit this program
.features        List osquery's features and their statuses
.headers ON|OFF  Turn display of headers on or off
.help            Show this message
.mode MODE       Set output mode where MODE is one of:
                   csv      Comma-separated values
                   column   Left-aligned columns see .width
                   line     One value per line
                   list     Values delimited by .separator string
                   pretty   Pretty printed SQL results (default)
.nullvalue STR   Use STRING in place of NULL values
.print STR...    Print literal STRING
.quit            Exit this program
.schema [TABLE]  Show the CREATE statements
.separator STR   Change separator used by output mode
.socket          Show the osquery extensions socket path
.show            Show the current values for various settings
.summary         Alias for the show meta command
.tables [TABLE]  List names of tables
.width [NUM1]+   Set column widths for "column" mode
.timer ON|OFF      Turn the CPU timer measurement on or off
osquery>

```

以后就算忘记了某些命令也可以监控系统了，👍。