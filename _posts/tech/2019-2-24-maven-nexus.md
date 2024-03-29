---
layout: post
category: Tech
title: 如何搭建 Maven 私服
tags: [java,ops]
excerpt: 项目开发中为了可以重用工具类等，可通过搭建 Maven私服 的方式来简化开发和依赖管理
---

{% include JB/setup %}

[Sonatype Nexus](https://www.sonatype.com/nexus/repository-oss) 是一个免费的包管理器，包含Maven/Java, npm, NuGet, RubyGems等，这里我们通过它来管理 jar 包。

## 安装 配置Sonatype Nexus

```sh
# 下载
# https://www.sonatype.com/nexus/repository-oss-download
wget https://sonatype-download.global.ssl.fastly.net/repository/repositoryManager/3/nexus-3.14.0-04-unix.tar.gz
tar -xzf nexus-3.14.0-04-unix.tar.gz
mv nexus-3.14.0-04 /opt/nexus

# 创建 nexus.service
vim /etc/systemd/system/nexus.service

####
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
####

# 启动 service
sudo systemctl daemon-reload
sudo systemctl enable nexus.service
sudo systemctl start nexus.service
```

编辑Maven的配置文件`settings.xml`(Linux中路径为/usr/share/maven/conf/settings.xml)，添加服务器的账号密码(默认为账号为admin，密码为admin123)。
```xml
<server>
      <id>maven-snapshots</id>
      <username>${NAME}</username>
      <password>${PASSWORD}</password>
</server>
```

在项目的pom.xml中添加服务器地址（即可通过 `mvn deploy` 发布jar包）。
```xml
<distributionManagement>
        <repository>
            <id>maven-releases</id>
            <name>Maven Release Repository</name>
            <url>http://${IP}:${PORT}/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>maven-snapshots</id>
            <name>Maven Snapshot Repository</name>
            <url>http://${IP}:${PORT}/repository/maven-snapshots/</url>
        </snapshotRepository>
</distributionManagement>
```

## 常见问题

#### 1. Cannot run program "gpg"

安装 `GnuPG` 即可, 如 "macOS" 安装: `brew install gnupg`

#### 2. 签名

Gpg服务器列表:

[SKS Keyservers: Status pages](ttps://sks-keyservers.net/status/)

上传公钥到Gpg服务器:

```
gpg --keyserver pgpkeys.co.uk --send-key [userid]
```

Gpg签名时报`signing failed: Inappropriate ioctl for device`错误，执行：

```
export GPG_TTY=$(tty)
```

#### 3. 仓库镜像配置

官方文档: [Using Mirrors for Repositories](https://maven.apache.org/guides/mini/guide-mirror-settings.html#using-mirrors-for-repositories)

`mirrorOf` 根据 `repository.id` 进行配置, 项目内部配置仓库信息后，mirrorOf 中配置该 `id` 即可让镜像只对该仓库生效，`!id` 即为忽略该仓库

```xml
    <settings>
      ...
      <mirrors>
        <mirror>
          <id>other-mirror</id>
          <name>Other Mirror Repository</name>
          <url>https://other-mirror.repo.other-company.com/maven2</url>
          <mirrorOf>central</mirrorOf>
        </mirror>
      </mirrors>
      ...
    </settings>
```