---
layout: post
category: Tech
title: 编译 OpenJDK 源码
tags: [java, source]
excerpt: 了解 JDK 的第一步就是了解如何编译 JDK
---

{% include JB/setup %}

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 配置](#1-配置)
- [2. 编译](#2-编译)
- [3. 验证](#3-验证)
- [4. 问题](#4-问题)
  - [1. unzip](#1-unzip)
  - [2. zip](#2-zip)
  - [3. boot jdk](#3-boot-jdk)
  - [4. systemtap-sdt](#4-systemtap-sdt)
  - [5. X11](#5-x11)
  - [6. cups](#6-cups)
  - [7. font](#7-font)
  - [8. alsa](#8-alsa)

<!-- /code_chunk_output -->

测试环境:

* Ubuntu 16.04
* gcc 5.4.0
* jdk11-1ddf9a99e4ad

下载 jdk11 的源代码: 

``` sh
mkdir jdk11 && cd jdk11
# http://hg.openjdk.java.net/jdk/jdk11/
wget http://hg.openjdk.java.net/jdk/jdk11/archive/tip.tar.gz
tar -xzf tip.tar.gz
cd jdk11-1ddf9a99e4ad/
```

通过 README 文档可得构建文档为:

> 
>  * doc/building.html   (html version)
> * doc/building.md     (markdown version)

阅读 `doc/building.md`, 可总结为一下四步:

1. Run configure: `bash configure`
2. Run make: `make images`
3. Verify your newly built JDK: `./build/*/images/jdk/bin/java -version`
4. Run basic tests: `make run-test-tier1`(需要 JTReg)

安装编译基础工具:

```
sudo apt-get install build-essential
```

参考文档, 配置运行模式为`server`模式, 启用`debug`和`dtrace`:

```sh
bash configure --enable-debug --with-jvm-variants=server --enable-dtrace
```

### 1. 配置

运行命令:

```
bash configure --enable-debug --with-jvm-variants=server --enable-dtrace --with-boot-jdk=$HOME/jdk11/jdk-11.0.2
```

结果:

```
====================================================
A new configuration has been successfully created in
/home/cofcool/jdk11/jdk11-1ddf9a99e4ad/build/linux-x86_64-normal-server-fastdebug
using configure arguments '--enable-debug --with-jvm-variants=server --enable-dtrace --with-boot-jdk=/home/cofcool/jdk11/jdk-11.0.2'.

Configuration summary:
* Debug level:    fastdebug
* HS debug level: fastdebug
* JVM variants:   server
* JVM features:   server: 'aot cds cmsgc compiler1 compiler2 dtrace epsilongc g1gc graal jfr jni-check jvmci jvmti management nmt parallelgc serialgc services vm-structs'
* OpenJDK target: OS: linux, CPU architecture: x86, address length: 64
* Version string: 11-internal+0-adhoc.cofcool.jdk11-1ddf9a99e4ad (11-internal)

Tools summary:
* Boot JDK:       openjdk version "11.0.2" 2019-01-15 OpenJDK Runtime Environment 18.9 (build 11.0.2+9) OpenJDK 64-Bit Server VM 18.9 (build 11.0.2+9, mixed mode)  (at /home/cofcool/jdk11/jdk-11.0.2)
* Toolchain:      gcc (GNU Compiler Collection)
* C Compiler:     Version 5.4.0 (at /usr/bin/gcc)
* C++ Compiler:   Version 5.4.0 (at /usr/bin/g++)

Build performance summary:
* Cores to use:   2
* Memory limit:   7965 MB
```

### 2. 编译

运行命令:

```
make images
```

结果:

```
Building target 'images' in configuration 'linux-x86_64-normal-server-fastdebug'
Compiling 8 files for BUILD_TOOLS_LANGTOOLS
Creating hotspot/variant-server/tools/adlc/adlc from 13 file(s)

...

Creating support/demos/image/jfc/SampleTree/SampleTree.jar
Creating support/demos/image/jfc/TableExample/TableExample.jar
Creating support/demos/image/jfc/TransparentRuler/TransparentRuler.jar
Creating jdk image
Stopping sjavac server
Finished building target 'images' in configuration 'linux-x86_64-normal-server-fastdebug'
```

### 3. 验证

运行命令:

```sh
build/linux-x86_64-normal-server-fastdebug/images/jdk/bin/java -version
```

结果:

```
openjdk version "11-internal" 2018-09-25
OpenJDK Runtime Environment (fastdebug build 11-internal+0-adhoc.cofcool.jdk11-1ddf9a99e4ad)
OpenJDK 64-Bit Server VM (fastdebug build 11-internal+0-adhoc.cofcool.jdk11-1ddf9a99e4ad, mixed mode)
```

如需测试，需在编译配置期间配置`JTReg`，添加 `--with-jtreg=<path>`, 运行 `make run-test-tier1` 进行测试。

### 4. 问题

#### 1. unzip

错误:

> configure: error: Could not find required tool for UNZIP

解决:

```
sudo apt install unzip
```


#### 2. zip

错误:

> checking for zip... no

解决:

```
sudo apt install zip
```

#### 3. boot jdk

错误:

> configure: Could not find a valid Boot JDK. You might be able to fix this by running 'sudo apt-get install openjdk-8-jdk'.

根据提示安装"jdk 8"

```
sudo apt-get install openjdk-8-jdk
```

但是仍旧报错，提示"Boot JDK"版本需为 10 或 11

> configure: Potential Boot JDK found at /usr/lib/jvm/java-1.8.0-openjdk-amd64 is incorrect JDK version (openjdk version "1.8.0_222"); ignoring
> configure: (Your Boot JDK version must be one of: 10 11)
> configure: Could not find a valid Boot JDK. You might be able to fix this by running 'sudo apt-get install openjdk-8-jdk'.
> configure: This might be fixed by explicitly setting --with-boot-jdk

下载配置"openjdk-11.0.2"即可

```
wget https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_linux-x64_bin.tar.gz
tar -xzf openjdk-11.0.2_linux-x64_bin.tar.gz
```

配置添加`--with-boot-jdk`即可

```
bash configure --enable-debug --with-jvm-variants=server --enable-dtrace --with-boot-jdk=$HOME/jdk11/jdk-11.0.2
```

#### 4. systemtap-sdt

错误:

> configure: error: Cannot enable dtrace with missing dependencies. See above. You might be able to fix this by running 'sudo apt-get install systemtap-sdt-dev'.

解决:

```
sudo apt-get install systemtap-sdt-dev
```

#### 5. X11

错误:

> configure: error: Could not find all X11 headers (shape.h Xrender.h XTest.h Intrinsic.h). You might be able to fix this by running 'sudo apt-get install libx11-dev libxext-dev libxrender-dev libxtst-dev libxt-dev'.

解决:

```
sudo apt-get install libx11-dev libxext-dev libxrender-dev libxtst-dev libxt-dev
```

#### 6. cups

错误:

> checking cups/ppd.h presence... configure: error: Could not find cups! You might be able to fix this by running 'sudo apt-get install libcups2-dev'.

解决:

```
sudo apt-get install libcups2-dev
```

#### 7. font

错误:

> configure: error: Could not find fontconfig! You might be able to fix this by running 'sudo apt-get install libfontconfig1-dev'

解决:

```
sudo apt-get install libfontconfig1-dev
```

#### 8. alsa

错误:

> configure: error: Could not find alsa! You might be able to fix this by running 'sudo apt-get install libasound2-dev'.

解决:

```
sudo apt-get install libasound2-dev
```