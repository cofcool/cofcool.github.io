---
layout: post
category : Tech
title : 基于 GraalVM 开发一款常用工具箱
tags : [java, graalvm]
excerpt: 想不想使用最新 Java 技术开发一款属于你自己的小工具集合呢？
---
{% include JB/setup %}

2022年底受疫情影响，有了一些空闲时间，思维便开始发散起来，在想要做个什么好了。正好需要把 Trello 里的内容导入到 Logseq，突然想到为什么不利用最新的 Java 技术来开发，这样既可以了解最新的 Java 特性，也可以实现导入目的。于是 [MyToolbox](https://github.com/cofcool/my-toolbox) 就呱呱坠地了。

建立项目后，先开始思索项目整体架构应该是什么样的：

1. 命令行程序
2. 添加新功能要方便
3. 保持精简小巧，避免依赖过多
4. 统一处理参数解析、日志打印等和命令实现无关的操作
5. 跨平台

这样大致可以确定程序流程和架构了。

<svg xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" contentstyletype="text/css" height="359px" preserveAspectRatio="none" style="width:257px;height:359px;background:#FFFFFF;" version="1.1" viewBox="0 0 257 359" width="257px" zoomAndPan="magnify"><defs></defs><g><text fill="#000000" font-family="sans-serif" font-size="14" font-weight="bold" lengthAdjust="spacing" textLength="56" x="99" y="33.5352">执行流程</text><ellipse cx="197" cy="57.4883" fill="#222222" rx="10" ry="10" style="stroke:#222222;stroke-width:1.0;"></ellipse><rect fill="#F1F1F1" height="34.1328" rx="12.5" ry="12.5" style="stroke:#181818;stroke-width:0.5;" width="72" x="161" y="87.4883"></rect><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacing" textLength="48" x="175" y="109.0898">输入命令</text><path d="M10,138.377 L10,178.998 A0,0 0 0 0 10,178.998 L135,178.998 A0,0 0 0 0 135,178.998 L135,162.6875 L155,158.6875 L135,154.6875 L135,148.377 L125,138.377 L10,138.377 A0,0 0 0 0 10,138.377 " fill="#FEFFDD" style="stroke:#181818;stroke-width:0.5;"></path><path d="M125,138.377 L125,148.377 L135,148.377 L125,138.377 " fill="#FEFFDD" style="stroke:#181818;stroke-width:0.5;"></path><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="104" x="16" y="155.9453">解析、验证参数，</text><text fill="#000000" font-family="sans-serif" font-size="13" lengthAdjust="spacing" textLength="56" x="16" y="171.2559">提取命令;</text><rect fill="#F1F1F1" height="34.1328" rx="12.5" ry="12.5" style="stroke:#181818;stroke-width:0.5;" width="84" x="155" y="141.6211"></rect><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacing" textLength="60" x="169" y="163.2227">解析参数等</text><rect fill="#F1F1F1" height="34.1328" rx="12.5" ry="12.5" style="stroke:#181818;stroke-width:0.5;" width="72" x="161" y="198.998"></rect><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacing" textLength="48" x="175" y="220.5996">匹配命令</text><rect fill="#F1F1F1" height="34.1328" rx="12.5" ry="12.5" style="stroke:#181818;stroke-width:0.5;" width="72" x="161" y="274.4063"></rect><text fill="#000000" font-family="sans-serif" font-size="12" lengthAdjust="spacing" textLength="48" x="175" y="296.0078">执行命令</text><ellipse cx="197" cy="338.5391" fill="none" rx="10" ry="10" style="stroke:#222222;stroke-width:1.5;"></ellipse><line style="stroke:#222222;stroke-width:2.5;" x1="190.8128" x2="203.1872" y1="332.3519" y2="344.7262"></line><line style="stroke:#222222;stroke-width:2.5;" x1="203.1872" x2="190.8128" y1="332.3519" y2="344.7262"></line><line style="stroke:#181818;stroke-width:1.0;" x1="197" x2="197" y1="67.4883" y2="87.4883"></line><polygon fill="#181818" points="193,77.4883,197,87.4883,201,77.4883,197,81.4883" style="stroke:#181818;stroke-width:1.0;"></polygon><line style="stroke:#181818;stroke-width:1.0;" x1="197" x2="197" y1="121.6211" y2="141.6211"></line><polygon fill="#181818" points="193,131.6211,197,141.6211,201,131.6211,197,135.6211" style="stroke:#181818;stroke-width:1.0;"></polygon><line style="stroke:#181818;stroke-width:1.0;" x1="197" x2="197" y1="175.7539" y2="198.998"></line><polygon fill="#181818" points="193,188.998,197,198.998,201,188.998,197,192.998" style="stroke:#181818;stroke-width:1.0;"></polygon><line style="stroke:#181818;stroke-width:1.0;" x1="197" x2="197" y1="233.1309" y2="274.4063"></line><polygon fill="#181818" points="193,264.4063,197,274.4063,201,264.4063,197,268.4063" style="stroke:#181818;stroke-width:1.0;"></polygon><text fill="#000000" font-family="sans-serif" font-size="11" lengthAdjust="spacing" textLength="44" x="201" y="254.5859">传入参数</text><line style="stroke:#181818;stroke-width:1.0;" x1="197" x2="197" y1="308.5391" y2="328.5391"></line><polygon fill="#181818" points="193,318.5391,197,328.5391,201,318.5391,197,322.5391" style="stroke:#181818;stroke-width:1.0;"></polygon></g></svg>

因此需要实现的功能包括：

1. 程序入口函数
2. 参数解析器
3. 日志管理
4. 定义命令接口
5. 实现命令接口
6. 打包为可执行文件

前 5 步为具体的程序实现，就不具体说明了，`Tool` 接口定义如下，其它更多细节详情可查看源码。

```java
public interface Tool {

    // 名称
    ToolName name();

    // 执行
    void run(Args args) throws Exception;

    // 帮助信息
    Args config();
}
```

重点来看看打包环节，看 GraalVM 如何大显身手，可执行文件如何摆脱对 Java 的依赖的。

GraalVM 提供了 `org.graalvm.buildtools:native-maven-plugin` 插件构建原生可执行文件。构建流程：

1. 分析编译后的代码
2. 获取反射、资源文件、JNI 等信息
3. 根据配置信息构建原生可执行文件

运行 `java -agentlib:native-image-agent=config-merge-dir=./src/main/resources/META-INF/native-image/config -jar ./target/my-toolbox-fat.jar` 生成配置信息，`mvn package` 即可生成原生可执行文件。

🎉🎉🎉，就这样我们的工具箱就完成了。

运行示例：

```shell
$ ~ mytool
ERROR: Please check tool name
About: CofCool@ToolBox v1.0.7
Example: --tool=demo --path=tmp
Help: --help={COMMAND}, like: --help=rename
Tools:
    rename: rename file conveniently
    trelloLogseqImporter: read trello backup json file and convert to logseq md file
    shell: run shell command
    dirWebServer: start a simple web directory server
    gitCommits2Log: generate changelog file from git commit log
    kindle: read kindle clipboard file and convert to md file
    json2POJO: convert json structure to POJO class
    link2Tool: convert link file to md
    converts: some simple utilities about string, like base64 encode
```

计算“abc”的 MD5 值：

```shell
$ ~ mytool --md5=abc
Start run converts
900150983cd24fb0d6963f7d28e17f72
```

**参考资料**

* https://docs.oracle.com/en/learn/understanding-reflection-graalvm-native-image/index.html#conclusions