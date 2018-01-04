---
layout: post
category : Tech
title : Android Gradle编译的一些问题
tags : [android]
---
{% include JB/setup %}

目录:
<!-- @import "[TOC]" {cmd="toc" depthFrom=3 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 库引用指令变为api, implementation](#1-库引用指令变为api-implementation)
* [2. 同一库不同版本问题](#2-同一库不同版本问题)
* [3. 引用其它项目时，报Unable to resolve dependency for ':app@debug/compileClasspath'错误](#3-引用其它项目时报unable-to-resolve-dependency-for-appdebugcompileclasspath错误)

<!-- /code_chunk_output -->


### 1. 库引用指令变为api, implementation

Android Gradle Tools 升级3.0之后新增`api`, `implementation`函数，compile被弃用。api和compile用法相同，implementation会将该依赖隐藏在内部，而不对外部公开，也就是说其他项目无法通过依赖该项目来依赖`implementation`编译的依赖。

### 2. 同一库不同版本问题

如果项目引用的第三方库和项目本身对同一库的不同版本有引用,使用`gradle -q app:dependencies`可列出项目和引用库的依赖情况，找到发生冲突的依赖，然后通过`exclude`来排除该第三方库对该库的引用。

```groovy
implementation('com.nononsenseapps:filepicker:4.1.0') {
    exclude group:'com.android.support'
}
```

### 3. 引用其它项目时，报Unable to resolve dependency for ':app@debug/compileClasspath'错误

如果引用其它项目，需要修改该项目的`build.gradle`，把application替换为library。

```groovy
apply plugin: 'com.android.library'
```
