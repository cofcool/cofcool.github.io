---
layout: post
category : Tech
title : Python虚拟环境搭建
tags : [python]
---
{% include JB/setup %}

目录:
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 安装](#1-安装)
* [2. 使用](#2-使用)
	* [2.1 创建虚拟环境](#21-创建虚拟环境)
	* [2.2 激活虚拟环境](#22-激活虚拟环境)
	* [2.3 退出虚拟环境](#23-退出虚拟环境)
* [3. 示例](#3-示例)

<!-- /code_chunk_output -->


## 1. 安装

```sh
pip insall virtualenv
```

## 2. 使用

### 2.1 创建虚拟环境

执行`virtualenv 环境名`即可， 如`virtualenv django`

### 2.2 激活虚拟环境

执行`source ${PATH}/bin/activate`，${PATH}为虚拟环境根目录

### 2.3 退出虚拟环境

执行`deactivate`，如果想要删除虚拟环境，只需删除虚拟环境目录即可。

## 3. 示例

```sh
~ > virtualenv django
~ > cd django
~ > tree -L 2
.
├── bin
│   ├── activate
│   ├── activate.csh
│   ├── activate.fish
│   ├── activate_this.py
│   ├── django-admin
│   ├── django-admin.py
│   ├── django-admin.pyc
│   ├── easy_install
│   ├── easy_install-2.7
│   ├── pip
│   ├── pip2
│   ├── pip2.7
│   ├── python -> python2.7
│   ├── python-config
│   ├── python2 -> python2.7
│   ├── python2.7
│   └── wheel
├── include
│   └── python2.7 -> /usr/local/Cellar/python/2.7.14_1/Frameworks/Python.framework/Versions/2.7/include/python2.7
├── lib
│   └── python2.7
└── pip-selfcheck.json

5 directories, 18 files
~ django > source ./bin/activate
django ~ django > pip install django
Collecting django
  Downloading Django-1.11.12-py2.py3-none-any.whl (6.9MB)
    100% |████████████████████████████████| 7.0MB 2.1MB/s
Collecting pytz (from django)
  Downloading pytz-2018.4-py2.py3-none-any.whl (510kB)
    100% |████████████████████████████████| 512kB 2.6MB/s
Installing collected packages: pytz, django
Successfully installed django-1.11.12 pytz-2018.4
django ~ django > deactivate
~ >
```
