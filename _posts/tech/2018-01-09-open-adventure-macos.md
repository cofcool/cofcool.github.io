---
layout: post
category : Tech
title : 在macOS上玩巨洞冒险
tags : [game]
---
{% include JB/setup %}

无意中看到 [巨洞冒险：史上最有名的经典文字冒险游戏 ](https://linux.cn/article-9209-1.html)，发现这个游戏还挺有意思。想要在macOS上试试，但是并没有该文章并没有说明在macOS如何安装。大概看了看该游戏的源码，该代码同样可以在macOS上编译。

步骤如下：

#### 1. 依赖安装

```sh
pip3 install PyYAML
brew install libedit

# 如果没有该文件，可执行brew install pkg-config安装pkg-config
cd /usr/local/lib/pkgconfig
ln -s ../../Cellar/libedit/20170329-3.1/lib/pkgconfig/libedit.pc libedit.pc
```

#### 2. 代码编译

```sh
git clone https://gitlab.com/esr/open-adventure.git

cd open-adventure
vim make_dungeon.py

# 修改执行环境为python3，修改后如下
#!/usr/bin/env python3

# 编译
make

# 运行
./advent
```

我们开始愉快的游戏吧!!!

```
Welcome to Adventure!!  Would you like instructions?

> yes

Somewhere nearby is Colossal Cave, where others have found fortunes in
treasure and gold, though it is rumored that some who enter are never
seen again.  Magic is said to work in the cave.  I will be your eyes
and hands.  Direct me with commands of 1 or 2 words.  I should warn
you that I look at only the first five letters of each word, so you'll
have to enter "northeast" as "ne" to distinguish it from "north".
You can type "help" for some general hints.  For information on how
to end your adventure, scoring, etc., type "info".
			      - - -
This program was originally developed by Willie Crowther.  Most of the
features of the current program were added by Don Woods.

You are standing at the end of a road before a small brick building.
Around you is a forest.  A small stream flows out of the building and
down a gully.
```
