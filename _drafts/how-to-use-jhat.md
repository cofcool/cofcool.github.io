# 详解 jstat 等 jvm 调试工具

* jinfo
* jps
* jcmd
* jmap
* jstack
* jstat 
* jhsdb

## [jinfo](https://docs.oracle.com/javase/10/tools/jinfo.htm#JSWOR744)

生成指定线程的 Java 配置信息，注意：实验性质，未来可能会移除。

使用方式: 

```
jinfo [option] pid
```

参数:

-flag name

打印指定配置项的名字和值

-flag [+|-]name

开启或关闭指定布尔值的配置项

-flag name=value

配置指定配置项的值

-flags

打印虚拟机配置的配置项

-sysprops

打印 Java System Properties

-h or -help

打印帮助信息

例如:

```sh
jinfo -sysprops 5031

Java System Properties:
#Tue Dec 10 16:13:36 CST 2019
gopherProxySet=false
awt.toolkit=sun.lwawt.macosx.LWCToolkit
java.specification.version=11
sun.cpu.isalist=
sun.jnu.encoding=UTF-8
...
```

## [jps](https://docs.oracle.com/javase/10/tools/jps.htm#JSWOR733)

列出当前系统运行的 JVM 实例

使用方式:

```
jps [ -q ] [ -mlvV ][hostid ]

jps [ -help ]
```

参数:

-q

    Suppresses the output of the class name, JAR file name, and arguments passed to the main method, producing a list of only local JVM identifiers.
-mlvV

        -m displays the arguments passed to the main method. The output may be null for embedded JVMs.

        -l displays the full package name for the application's main class or the full path name to the application's JAR file.

        -v displays the arguments passed to the JVM.

        -V suppresses the output of the class name, JAR file name, and arguments passed to the main method, producing a list of only local JVM identifiers.

hostid

    The identifier of the host for which the process report should be generated. The hostid can include optional components that indicate the communications protocol, port number, and other implementation specific data. See Host Identifier.
-help

    Displays the help message for the jps command.
