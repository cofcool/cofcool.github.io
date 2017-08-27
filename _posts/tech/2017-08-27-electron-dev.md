---
layout: post
category : Tech
title : Electron笔记
tags : [electron, notes]
---
{% include JB/setup %}

目录:

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 环境配置](#1-环境配置)
* [2. 简单示例](#2-简单示例)
* [3. 打包发布](#3-打包发布)
	* [1. 打包](#1-打包)
	* [2. 发布](#2-发布)

<!-- /code_chunk_output -->


## 1. 环境配置

1. install
```
// install node.js, npm
sudo npm install -g electron
// if it is failed, change npm mirror
npm config set registry https://registry.npm.taobao.org
```

## 2. 简单示例

**electron-quick-start** [ Clone to try a simple Electron app ](https://github.com/electron/electron-quick-start)

files:
- package.json
- main.js
- index.html

1. 通过`npm init`初始化项目，按照提示创建`package.json`，添加配置项，如依赖等
2. 安装依赖 `npm install`
3. 编写基本逻辑

    `main.js`

    ```
    const electron = require('electron')
    // 初始化app组件
    const app = electron.app
    // 创建应用窗口
    const BrowserWindow = electron.BrowserWindow

    const path = require('path')
    const url = require('url')

    // 声明应用主窗口，为了防止窗口关闭后该变量被销毁，需要在合适的场景重新为该变量赋值
    let mainWindow

    function createWindow () {
      // 创建浏览器窗口
      mainWindow = new BrowserWindow({width: 800, height: 600})

      // 为主窗口设置主页
      mainWindow.loadURL(url.format({
        pathname: path.join(__dirname, 'index.html'),
        protocol: 'file:',
        slashes: true
      }))

      // 显示开发者工具视图
      mainWindow.webContents.openDevTools()

      // 当窗口关闭后触发closed事件
      mainWindow.on('closed', function () {
        // 销毁主窗口
        mainWindow = null
      })
    }

    // 当应用数据准备完成后触发
    app.on('ready', createWindow)

    // 退出应用触发
    app.on('window-all-closed', function () {
      // Mac OS X 关闭窗口时应用不需要退出，所以不需要销毁app
      if (process.platform !== 'darwin') {
        app.quit()
      }
    })

    app.on('activate', function () {
      // 应用重新获得焦点时，创建窗口，仅适用于Mac OS X
      if (mainWindow === null) {
        createWindow()
      }
    })
    ```
    `index.html`
    ```
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8">
        <title>Hello World!</title>
      </head>
      <body>
        <h1>Hello World!</h1>
        We are using Node.js <script>document.write(process.versions.node)</script>,
        Chromium <script>document.write(process.versions.chrome)</script>,
        and Electron <script>document.write(process.versions.electron)</script>.
      </body>
    </html>
    ```
4. 运行项目 `electron .`

## 3. 打包发布

### 1. 打包

```
sudo npm install -g asar
asar pack ${directory} app.asar
```

### 2. 发布

```
sudo npm install electron-packager -g
electron-packager ${directory} ${anem} --platform=linux --arch=x64 --arar=true  --prune --overwrite
```

***NOTE***:

1. npm安装包的时候如果有`-g`参数，虽然该包会加入到全局环境变量中，但是项目中并不会引用。
2. 使用`Atom`开发时，通过`terminal`插件运行`electron .`时，应用无法启动， 报错：
    ```
    app.on('ready', createWindw);
    ^

    TypeError: Cannot read property 'on' of undefined
    ```
    请使用系统中的`terminal`软件。
    [require("electron").app is undefined - stackoverflow](http://stackoverflow.com/questions/40664009/requireelectron-app-is-undefined-i-npm-installed-fresh-modules-unsure-what/40664925#40664925)
