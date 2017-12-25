---
layout: post
category : Tech
title : Windows实现定时自动锁屏
tags : [Windows]
---

目录：

<!-- @import "[TOC]" {cmd="toc" depthFrom=3 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1.  构建锁屏软件](#1-构建锁屏软件)
* [2. 建立计划任务](#2-建立计划任务)
	* [2.1 创建基本任务](#21-创建基本任务)
	* [2.2 配置计划任务](#22-配置计划任务)

<!-- /code_chunk_output -->


### 1.  构建锁屏软件

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Runtime.InteropServices;

namespace OneKeyLock
{
    class Program
    {
        static void Main(string[] args)
        {
            LockWorkStation();
        }

        [DllImport("user32.dll")]
        private static extern void LockWorkStation();
    }
}

```

使用VS创建一个C#控制台项目，编译以上代码，生成一个可执行软件。

### 2. 建立计划任务

#### 2.1 创建基本任务

![autolock](http://cofcool.net/imgs/autolock-01.png)

#### 2.2 配置计划任务

任务设为在用户登录时运行

![autolock](http://cofcool.net/imgs/autolock-02.jpg)

触发器配置如下图所示：

![autolock](http://cofcool.net/imgs/autolock-03.jpg)