---
layout: post
category : Tech
title : Atom使用
tags : [atom, notes]
---
{% include JB/setup %}

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 安装](#1-安装)
	* [1. 安装](#1-安装-1)
	* [2. 中文乱码解决](#2-中文乱码解决)
* [2. 配置](#2-配置)
	* [1. 代码片段](#1-代码片段)
	* [2. 导出已安装 package](#2-导出已安装-package)
	* [3. 常用 package](#3-常用-package)

<!-- /code_chunk_output -->


## 1. 安装

### 1. 安装

通过 [atom.io](https://atom.io)下载安装包即可。

***这篇笔记是在atom处于测试阶段时写的，后来整理的时候atom已经很完善了，我当初碰到的一些基本问题已修复了，例如中文显示异常，package无法安装等。以下内容仅是记录作用，并无实际意义,包括(中文乱码解决)。***

```
 sudo apt-get install nodejs-legacy
 sudo npm install -g atom-shell

 vim ~/.npmrc
 registry =https://registry.npm.taobao.org
 strict-ssl =false

/usr/local/bin/atom-shell -> /usr/local/lib/node_modules/atom-shell/run.bat

atom-shell@0.22.3-1 install /usr/local/lib/node_modules/atom-shell
node install.js

Downloading atom-shell-v0.22.3-linux-x64.zip
Error: socket hang up

Error: socket hang up
npm WARN This failure might be due to the use of legacy binary "node"
npm WARN For further explanations, please read
/usr/share/doc/nodejs/README.Debian

npm ERR! weird error 8
npm ERR! not ok code 0


npm ERR! Error: CERT_UNTRUSTED
gyp ERR! not ok
npm WARN This failure might be due to the use of legacy binary "node"
npm WARN For further explanations, please read
/usr/share/doc/nodejs/README.Debian

npm ERR! weird error 1
npm ERR! not ok code 0
```

### 2. 中文乱码解决

最近在Ubuntu 14.04 上使用Atom时，发现打开中文名的文件或是含有中文的文件时中文都是方块，在设置中也没有一个更改语言或是类似的选项。试着上网查了一下，发现很多人都遇到了这个问题，解决方法如下：

1. 安装中文字体
   ```
   # 文泉驿字体
   sudo apt-get install ttf-wqy-*
   ```
   设置

   ![image]({{ site.url }}/public/upload/images/shot0051.png)

   如下:
   ```
   /*
    * Your Stylesheet
    *
    * This stylesheet is loaded when Atom starts up and is reloaded automatically
    * when it is changed.
    *
    * If you are unfamiliar with LESS, you can read more about it here:
    * http://www.lesscss.org
    */

    /* 字体 */
    @mono-font-family: "ubuntu mono", "Microsoft YaHei","WenQuanYi Micro Hei", sans-serif;
    @font-family: "ubuntu", "Microsoft YaHei","WenQuanYi Micro Hei", sans-serif;

   .tree-view, .current-path, .title, .tooltip {
       font-family: @font-family;
   }

   .editor {
       font-family:  @mono-font-family;
   }

   .editor .cursor {

   }

   .markdown-preview {
       font-family: @font-family;
   }
   ```

## 2. 配置

### 1. 代码片段

1. 打开配置文件
    ![imgae]({{ site.url }}/public/upload/images/0066.PNG)
2. 添加代码片段
    ```
    '.html':                   # 文件后缀名
      'my td':                # 描述
        'prefix': "td"       # 自动补全时输入的字符
        'body': "<td>$1</td>" # 具体内容
    ```

   更多可参考[atom帮助手册](https://atom.io/docs/latest/using-atom-basic-customization)。
### 2. 导出已安装 package

```
# 导出安装包列表
apm list  -p -b > packages.txt

# 根据列表安装
apm install `cat packages.txt`
```

### 3. 常用 package

|            名字             |               描述               |
| :-----------------------: | :----------------------------: |
|    activate-power-mode    |           输入时激发酷炫效果            |
|        atom-ctags         |             ctags              |
|         git-plus          |              git               |
|       markdown-pdf        |         markdown导出为pdf         |
| markdown-preview-enhanced | markdown预览，支持mermaid,plantuml等 |
|   markdown-preview-plus   |           markdown预览           |
|       markdown-toc        |              生成目录              |
|      markdown-writer      |          markdown辅助工具          |
|          minimap          |         文件一侧显示minimap          |
|          nuclide          |      facebook出品的react开发工具      |
|  platformio-ide-terminal  |            terminal            |
|         todo-show         |             todo工具             |
|   tree-view-git-status    |       tree-view中显示git状态        |
|     typewriter-sounds     |         输入文本时发出打字机的声音          |
|         vim-mode          |            激活vim模式             |
