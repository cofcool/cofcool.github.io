---
layout: post
category : Tech
title : Windows实现定时自动锁屏
tags : [Windows]
excerpt: Windows使用脚本实现自动锁频。
---

目录：

<!-- @import "[TOC]" {cmd="toc" depthFrom=3 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

- [1.  构建锁屏软件](#1-构建锁屏软件)
- [2. 建立计划任务](#2-建立计划任务)
  - [2.1 创建基本任务](#21-创建基本任务)
  - [2.2 配置计划任务](#22-配置计划任务)

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


经过网友 [@qq8-------6](https://github.com/qq8-------6) 提醒，可直接通过命令行运行，不需要编译成可执行文件，详情参考 [【Windows实现定时自动锁屏】小小改进建议](https://github.com/cofcool/cofcool.github.io/issues/1)