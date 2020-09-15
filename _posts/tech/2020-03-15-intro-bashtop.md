---
layout: post
category : Tech
title : 酷炫非凡的系统资源监控利器 Bashtop
tags : [ops]
excerpt: Bashtop 是一款功能强大界面酷炫的系统监控软件，可用于 Linux 等类 Unix 系统，主要功能由 Shell 和 Python 开发。
---
{% include JB/setup %}

环境:

* Ubuntu 20.04
* Bashtop v0.9.25

## 1. 简要介绍

[Bashtop](https://github.com/aristocratos/bashtop) 是一款功能强大界面酷炫的系统监控软件，可用于“Linux”等类"Unix"系统。主要功能由 `Shell` 和 `Python` 开发，数据收集依赖 [psutil: Cross-platform lib for process and system monitoring in Python](https://github.com/giampaolo/psutil)(未安装时调用 [bashtop.psutil.py](https://github.com/aristocratos/bashtop/blob/master/src/bashtop.psutil.py))，[bashtop](https://github.com/aristocratos/bashtop/blob/master/bashtop) 脚本负责界面渲染和操作逻辑等。

> Resource monitor that shows usage and stats for processor, memory, disks, network and processes.

![main menu](https://raw.githubusercontent.com/aristocratos/bashtop/master/Imgs/menu.png)

![main ui](https://raw.githubusercontent.com/aristocratos/bashtop/master/Imgs/main.png)


## 2. 安装

常见“Linux”系统的软件源已经包含了 `Bashtop`，具体安装如下：

**Fedora**

```sh
sudo dnf install bashtop
```

**CentOS 8**

```sh
dnf config-manager --set-enabled PowerTools
dnf install epel-release
dnf install bashtop
```

**apt-get**

```sh
# Ubuntu 20.10 官方源已包含，低版本系统需添加 PPA
sudo add-apt-repository ppa:bashtop-monitor/bashtop
sudo apt update
sudo apt install bashtop

# 如果出现 Error: Missing python3 psutil module! 表明缺少 psutil 模块，安装即可
sudo apt install python3-psutil
```

其它环境可参考官方安装文档 [Installation](https://github.com/aristocratos/bashtop#installation)。

## 3. 使用

配置文件根目录为 `$HOME/.config/bashtop`，`bashtop.cfg`为配置文件，可配置主题，刷新时间间隔，是否启用 psutil 以及显示样式等。

```sh
#* Color theme, looks for a .theme file in "$HOME/.config/bashtop/themes" and "$HOME/.config/bashtop/user_themes"
#* Should be prefixed with either "themes/" or "user_themes/" depending on location, "Default" for builtin default theme
color_theme="Default"

#* Update time in milliseconds, increases automatically if set below internal loops processing time, recommended 2000 ms or above for better sample times for graphs
update_ms="2500"

#* Processes sorting, "pid" "program" "arguments" "threads" "user" "memory" "cpu lazy" "cpu responsive"
#* "cpu lazy" updates sorting over time, "cpu responsive" updates sorting directly
proc_sorting="cpu lazy"

#* Reverse sorting order, "true" or "false"
proc_reversed="false"

#* Show processes as a tree
proc_tree="false"

#* Check cpu temperature, only works if "sensors", "vcgencmd" or "osx-cpu-temp" commands is available
check_temp="true"

#* Draw a clock at top of screen, formatting according to strftime, empty string to disable
draw_clock="%X"

#* Update main ui when menus are showing, set this to false if the menus is flickering too much for comfort
background_update="true"

#* Custom cpu model name, empty string to disable
custom_cpu_name=""

#* Enable error logging to "$HOME/.config/bashtop/error.log", "true" or "false"
error_logging="true"

#* Show color gradient in process list, "true" or "false"
proc_gradient="true"

#* If process cpu usage should be of the core it's running on or usage of the total available cpu power
proc_per_core="false"

#* Optional filter for shown disks, should be names of mountpoints, "root" replaces "/", separate multiple values with space
disks_filter=""

#* Enable check for new version from github.com/aristocratos/bashtop at start
update_check="true"

#* Enable graphs with double the horizontal resolution, increases cpu usage
hires_graphs="false"

#* Enable the use of psutil python3 module for data collection, default on OSX
use_psutil="true"
```

内置主题参考 [themes](https://github.com/aristocratos/bashtop/tree/master/themes)。