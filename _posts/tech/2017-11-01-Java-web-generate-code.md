---
layout: post
category : Tech
title : Java Web开发时快速生成模版代码
tags : [java, notes]
---
{% include JB/setup %}


环境: Spring MVC + Mybatis

在Java Web开发中，需要编写大量重复的代码，虽然不可缺少但是一一编写费事且无聊，是对生命的极其浪费啊，怎么能忍！

1. 使用[MyBatis Generator](http://www.mybatis.org/generator/)生成基础的单表DAO文件。
2. 做一次只有操作单表的Controller，Service的提交
3. 通过Git提取Patch，文本替换来生成新的Patch文件, 最后合并到代码分支。


还有个更好的方法就是修改Generator的源代码，根据需求把项目的相关代码一并生成。这里我们介绍第一种方法，因为简单快速！


<!-- @import "[TOC]" {cmd="toc" depthFrom=3 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 生成DAO](#1-生成dao)
* [2. 单表操作的提交](#2-单表操作的提交)
* [3. 生成操作相关表的patch文件](#3-生成操作相关表的patch文件)
* [4. 合并patch到代码分支](#4-合并patch到代码分支)
* [5. 修改产生的错误](#5-修改产生的错误)

<!-- /code_chunk_output -->


### 1. 生成DAO

1. 从Github克隆代码 `https://github.com/mybatis/generator.git`
2. 编译mybatis-generator-core: `mvn jar:jar `
3. 运行`java -jar mybatis-generator-core-x.x.x.jar -configfile generatorConfig.xml -overwrite`

generatorConfig.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <classPathEntry
            location="${MySQL驱动jar包路径}"/>

    <context id="${id}" targetRuntime="MyBatis3">

        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="true"/>
        </commentGenerator>

        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="${数据库路径}"
                        userId="${用户名}"
                        password="${密码}">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <javaModelGenerator targetPackage="${Bean的包名}" targetProject="src">
            <property name="enableSubPackages" value="false"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <sqlMapGenerator targetPackage="${生成Mapper的xml文件的包名}" targetProject="src">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>

        <javaClientGenerator type="XMLMAPPER" targetPackage="${Mapper类的包名}" targetProject="src">
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>


        <table tableName="${表名}"  domainObjectName="${Bean的名字}" />

    </context>
</generatorConfiguration>
```



### 2. 单表操作的提交

1. 创建Controller，Service
2. 提交代码
3. 生成patch: `git format-patch HEAD^`

### 3. 生成操作相关表的patch文件

执行以下代码:

```java
package test;

import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class GenerateCode {

    private static final String PATCH_PATH = ${PATCH_PATH};

    private static final String PATCH_BASE_PATH = {PATCH_BASE_PATH};



    public static void main(String[] args) {
        // NOTICE: 定义新的内容
        List<String[]> allKeys = new ArrayList<>();
        allKeys.add(new String[]{"表前缀", "表名", "文字描述"});

        try {
            GenerateCode generateCode = new GenerateCode();
            generateCode.setAllKeys(allKeys);
            generateCode.generatePatch();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }


    private BufferedReader reader;

    private List<String[]> allKeys;

    public GenerateCode() throws FileNotFoundException {
        this.reader = getBufferedReader();
    }

    public void generatePatch() throws IOException {
        File file = new File(PATCH_BASE_PATH);
        file.mkdirs();

        for (String[] keys: allKeys){
            reader = getBufferedReader();
            replacePatch(keys);
        }
    }

    private void replacePatch(String[] keys) throws IOException {
        StringBuilder newStrs = new StringBuilder();

        String lineStr = null;
        while ((lineStr = reader.readLine()) != null) {

            newStrs.append(replaceStr(lineStr, keys)).append("\n");
        }

        writePatch(newStrs.toString(), keys[1]);
    }

    private void writePatch(String string, String fileName) throws FileNotFoundException {
        FileOutputStream outputStream = new FileOutputStream(new File(PATCH_BASE_PATH + File.separator + fileName + ".patch"));

        PrintStream ps = new PrintStream(outputStream);
        ps.append(string);

        ps.close();
    }

    private BufferedReader getBufferedReader() throws FileNotFoundException {

        FileReader fileReader = new FileReader(new File(PATCH_PATH));

        return new BufferedReader(fileReader);
    }

    private String replaceStr(String originStr, String[] keys) {
        // NOTICE: 在此处定义要替换的内容，视具体情况而定
        return originStr.replace("表前缀", keys[0])
                        .replace("表名", keys[1])
                        .replace("创建的Java Bean的变量名", keys[1].toLowerCase())
                        .replace("文字描述", keys[2]);
    }

    public List<String[]> getAllKeys() {
        return allKeys;
    }

    public void setAllKeys(List<String[]> allKeys) {
        this.allKeys = allKeys;
    }
}
```

### 4. 合并patch到代码分支

执行`git am patch/*.patch`

### 5. 修改产生的错误



只要以上几步即可生成基础模版代码，是不是很方便快捷啊！
