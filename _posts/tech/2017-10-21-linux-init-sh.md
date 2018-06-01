---
layout: post
category : Tech
title : Ubuntu 配置备忘录
tags : [Ubuntu, notes]
---
{% include JB/setup %}

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 软件安装](#1-软件安装)
	* [1.1 常用软件](#11-常用软件)
	* [1.2 mysql](#12-mysql)
	* [1.3 sqlite3](#13-sqlite3)
	* [1.4 chrome](#14-chrome)
	* [1.5 VirtualBox](#15-virtualbox)
	* [1.6 oh-my-zsh](#16-oh-my-zsh)
	* [1.7 nodejs](#17-nodejs)
	* [1.8 常用Node工具包](#18-常用node工具包)
	* [1.9 Atom](#19-atom)
	* [1.10 Nvidia显卡驱动](#110-nvidia显卡驱动)
	* [1.11 ibus](#111-ibus)
	* [1.12 theme, icon](#112-theme-icon)
	* [1.13 gnome-shell extends](#113-gnome-shell-extends)
	* [1.14 开发文档查看工具](#114-开发文档查看工具)
	* [1.15 kodi](#115-kodi)
	* [1.16 Brackets](#116-brackets)
	* [1.17 folder-color](#117-folder-color)
	* [1.18 docker](#118-docker)
	* [1.19 php](#119-php)
	* [1.20 ocr](#120-ocr)
	* [1.21 video](#121-video)
	* [1.22 刻录](#122-刻录)
	* [1.23 思维导图](#123-思维导图)
	* [1.24 文件同步](#124-文件同步)
	* [1.25 gtk](#125-gtk)
	* [1.26 reader](#126-reader)
	* [1.27 virtual keyboard](#127-virtual-keyboard)
	* [1.28 Powerline](#128-powerline)
* [2. 配置](#2-配置)
	* [2.1 基本配置](#21-基本配置)
	* [2.2 Vim配置文件](#22-vim配置文件)
	* [2.3 PATH](#23-path)
	* [2.4 Powerline && Zsh](#24-powerline-zsh)
	* [2.5 ZSH](#25-zsh)
* [3. Apps](#3-apps)

<!-- /code_chunk_output -->


> 经常重装系统，每次都要安装一堆软件，而且不易记住，于以后方便，记录如下。

## 1. 软件安装

### 1.1 常用软件

```shell
sudo apt install samba python3 openjdk-8-jdk smplayer vlc bluefish qtcreator filezilla p7zip-full p7zip-rar ubuntu-restricted-extras blender wireshark kazam audacity git gitg subversion vim zsh docky gimp stardict meld thunderbird tree dconf-editor remmina deluge uget -y

# 剪切板
sudo apt install diodon

# ntfs
sudo apt install ntfs-3g exfat-utils

# 空格预览
sudo apt install unoconv
```

### 1.2 mysql

```sh
sudo apt install mysql-server mysql-client mycli mysql-workbench
```

### 1.3 sqlite3

```sh
sudo apt install sqlite3
```

### 1.4 chrome

```sh
sudo sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'

sudo apt-get install google-chrome-stable
```

### 1.5 VirtualBox

```sh
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
sudo apt-get update && sudo apt-get install virtualbox-5.1
```

### 1.6 oh-my-zsh

```sh
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
# zsh plugins
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git .oh-my-zsh/custom/plugins/zsh-syntax-highlighting
git clone git://github.com/zsh-users/zsh-autosuggestions .oh-my-zsh/custom/plugins/zsh-autosuggestions
```

### 1.7 nodejs

```sh
sudo apt install curl
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt install nodejs -y
```

### 1.8 常用Node工具包

```sh
react-native-cli
electron
webpack
vue-cli
create-react-app
```

### 1.9 Atom

```sh
# 导出安装包列表
apm list  -p -b > packages.txt

# 根据列表安装
apm install `cat packages.txt`
```


### 1.10 Nvidia显卡驱动

```sh
sudo apt install nvidia-375
```

### 1.11 ibus

```sh
sudo apt install ibus-anthy ibus-pinyin
```

### 1.12 theme, icon

```
sudo apt-add-repository ppa:numix/ppa
sudo apt-get update && sudo apt-get install numix-icon-theme-circle
```

### 1.13 gnome-shell extends

扩展路径:  `$HOME/.local/share/gnome-shell/extensions`

1. clipboard-indicator@tudmotu.com， 剪切版历史
2. RecentItems@bananenfisch.net ，最近打开的文件
3. system-monitor@paradoxxx.zero.gmail.com， 系统资源监视器
4. dash-to-dock@micxgx.gmail.com ，dock工具栏

```sh
# dependencies of system-monitor
sudo apt-get install gir1.2-gtop-2.0 gir1.2-networkmanager-2.0  gir1.2-clutter-1.0
```

### 1.14 开发文档查看工具

**1. Zeal**

Zeal is an offline documentation browser for software developers. like Dash of macOSThese docsets are generously provided by Dash. You can also create your own!

```
sudo apt-get install zeal
```
**2. DevDocs**

1. 使用Nativefier根据网址创建本地应用
  ```sh
  sudo npm install nativefier -g
  nativefier --name "DevDocs" "http://devdocs.io"
  ```
2. 创建.desktop文件


### 1.15 kodi

```
sudo add-apt-repository ppa:team-xbmc/ppa
sudo apt-get update && sudo apt-get install kodi
```

### 1.16 Brackets

### 1.17 folder-color

```
sudo add-apt-repository ppa:costales/folder-color
sudo apt-get update && sudo apt-get install folder-color
```

### 1.18 docker

### 1.19 php

### 1.20 ocr
sudo apt-get install ocrfeeder tesseract-ocr-chi*

### 1.21 video

```
sudo apt install winff ffmpeg
```

### 1.22 刻录

```
sudo apt install unetbootin
```

### 1.23 思维导图

```
sudo apt install freemind
```
[xmind](https://www.xmind.cn/)

### 1.24 文件同步

[freefilesync](https://www.freefilesync.org/)

### 1.25 gtk

```
sudo apt install dialog gnome-core-devel glade-gnome libglib* libgtk*
```

### 1.26 reader

```
sudo apt install fbreader chmsee
```

### 1.27 virtual keyboard

```
sudo apt install florence
```

### 1.28 Powerline

```
# git repository  git://github.com/Lokaltog/powerline
# sudo pip3 install --user git+git://github.com/powerline/powerline
sudo pip3 install powerline-status
# show status
pip3 show powerline-status
```

## 2. 配置

### 2.1 基本配置

```shell
# 调节亮度
echo 2100 > /sys/class/backlight/intel_backlight/brightness

# 关闭错误提示
vim /etc/default/apport # 0 -> 1

# 更改Grub2背景
cp ${PICTURE} /boot/grub/
```

### 2.2 Vim配置文件

```sh
# wget -qO- https://raw.github.com/ma6174/vim/master/setup.sh | sh -x
# https://github.com/spf13/spf13-vim
# spf13-vim
curl https://j.mp/spf13-vim3 -L -o - | sh
```

### 2.3 PATH

```
export PATH=$PATH:$HOME/bin/bin

# ANDROID PATH
export ANDROID_HOME=${PATH}/Android/Sdk
export PATH=${ANDROID_HOME}/tools:${ANDROID_HOME}/platform-tools:$PATH
```

### 2.4 Powerline && Zsh

```shell
# if the sh is bash, you can see the doucment from git repository.
if [ -f `which powerline-daemon` ]; then
  powerline-daemon -q
  POWERLINE_ZSH_CONTINUATION=1
  POWERLINE_ZSH_SELECT=1
  . /usr/local/lib/python3.5/dist-packages/powerline/bindings/zsh/powerline.zsh
fi
# if some symbol is ??, change the terminal font
git clone https://github.com/powerline/fonts.git && cd fonts
./install.sh
```

### 2.5 ZSH

```sh
# plugins
plugins=(git node npm autojump docker history wd zsh-autosuggestions zsh-syntax-highlighting debian git-extras jsontools)

# history date
HIST_STAMPS="yyyy-mm-dd""

```

## 3. Apps

```
├── DataGrip
├── DevDocs
├── Moeditor
├── Sublime3
├── VSCode
├── WebStorm
├── Android-studio
├── Atom
├── Eclipse
├── Gitkraken
├── idea-IC
├── idea-IU
├── jetty
├── Min
├── tomcat
└── xmind
```
