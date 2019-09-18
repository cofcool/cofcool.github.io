# 简述 Java Class 文件

目录:


<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 规范说明](#1-规范说明)
  - [1.1 magic](#11-magic)
  - [1.2 minor_version 和 major_version](#12-minor_version-和-major_version)
  - [1.3 constant_pool_count 和 constant_pool[]](#13-constant_pool_count-和-constant_pool)
  - [1.4 access_flags](#14-access_flags)
- [2. 实践](#2-实践)

<!-- /code_chunk_output -->


环境:

* JDK 11.0.1

本文简要概述 Java Class 文件格式，内容根据 "jvms11" 文档中的 `4. The class File Format`总结而来，主要内容是对 Class 文件格式的粗略描述。本文是为了加深我对该内容的理解而作，故行文为笔记格式，由于内容较为复杂，错误不可避免，大家可一起交流学习，另外也可参考 "The Java® Virtual Machine Specification Java SE 11 Edition"(如对 JVM 感兴趣可阅读)。

Class 文件结构定义如下:

```
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

下面我们来看看每个字段的含义和作用。

## 1. 规范说明

### 1.1 magic

`magic`标识文件为`class`文件, 值固定为`0xCAFEBABE`。

JDK 的 `com.sun.tools.javac.jvm.ClassFile`中定义`JAVA_MAGIC`:
```java
public final static int JAVA_MAGIC = 0xCAFEBABE;
```

### 1.2 minor_version 和 major_version

定义 `class` 文件版本。

历年来的 Class 文件版本对照:

Java SE | class file format version range
| -- | --
1.0.2 | 45.0 ≤ v ≤ 45.3
1.1 | 45.0 ≤ v ≤ 45.65535
1.2 | 45.0 ≤ v ≤ 46.0
1.3 | 45.0 ≤ v ≤ 47.0
1.4 | 45.0 ≤ v ≤ 48.0
5.0 | 45.0 ≤ v ≤ 49.0
6 | 45.0 ≤ v ≤ 50.0
7 | 45.0 ≤ v ≤ 51.0
8 | 45.0 ≤ v ≤ 52.0
9 | 45.0 ≤ v ≤ 53.0
10 | 45.0 ≤ v ≤ 54.0
11 | 45.0 ≤ v ≤ 55.0

`Java SE Platform` 需按照上表规定进行支持。

### 1.3 constant_pool_count 和 constant_pool[]

`constant_pool_count`为常量池中常量的数目, `constant_pool[]`为常量结构表, 包括"string, class and interface names, field names, and other constants that are referred to within the ClassFile structure and its substructures", `constant_pool`索引从`1`开始到`constant_pool_count - 1`。

### 1.4 access_flags

| Flag Name | Value | Interpretation 
| -- | -- | --
| ACC_PUBLIC | 0x0001 | Declared public; may be accessed from outside its package.
| ACC_FINAL | 0x0010 | Declared final; no subclasses allowed.
| ACC_SUPER | 0x0020 | Treat superclass methods specially when invoked by the invokespecial instruction.
| ACC_INTERFACE | 0x0200 | Is an interface, not a class.
| ACC_ABSTRACT | 0x0400 | Declared abstract; must not be instantiated.
| ACC_SYNTHETIC | 0x1000 | Declared synthetic; not present in the source code.
| ACC_ANNOTATION | 0x2000 | Declared as an annotation type.
| ACC_ENUM | 0x4000 | Declared as an enum type.
| ACC_MODULE | 0x8000 | Is a module, not a class or interface.

## 2. 实践

以`ClassTest`为例，源码如下:

```java
public class ClassTest {

    private String name;

    public ClassTest() {
    }

    public ClassTest(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        ClassTest test = new ClassTest("test");
        System.out.println(test.getName());
    }

}
```

"Class"文件的十六进制片段。

```
  1 00000000: cafe babe 0000 0037 0028 0a00 0900 1809  .......7.(......                                                                                               
  2 00000010: 0003 0019 0700 1a08 001b 0a00 0300 1c09  ................
  3 00000020: 001d 001e 0a00 0300 1f0a 0020 0021 0700  ........... .!..
  4 00000030: 2201 0004 6e61 6d65 0100 124c 6a61 7661  "...name...Ljava
  5 00000040: 2f6c 616e 672f 5374 7269 6e67 3b01 0006  /lang/String;...
  6 00000050: 3c69 6e69 743e 0100 0328 2956 0100 0443  <init>...()V...C
  7 00000060: 6f64 6501 000f 4c69 6e65 4e75 6d62 6572  ode...LineNumber
  8 00000070: 5461 626c 6501 0015 284c 6a61 7661 2f6c  Table...(Ljava/l
  9 00000080: 616e 672f 5374 7269 6e67 3b29 5601 0007  ang/String;)V...
 10 00000090: 6765 744e 616d 6501 0014 2829 4c6a 6176  getName...()Ljav
 11 000000a0: 612f 6c61 6e67 2f53 7472 696e 673b 0100  a/lang/String;..
 12 000000b0: 0773 6574 4e61 6d65 0100 046d 6169 6e01  .setName...main.
 13 000000c0: 0016 285b 4c6a 6176 612f 6c61 6e67 2f53  ..([Ljava/lang/S
 14 000000d0: 7472 696e 673b 2956 0100 0a53 6f75 7263  tring;)V...Sourc
 15 000000e0: 6546 696c 6501 000e 436c 6173 7354 6573  eFile...ClassTes
 16 000000f0: 742e 6a61 7661 0c00 0c00 0d0c 000a 000b  t.java..........
 17 00000100: 0100 0943 6c61 7373 5465 7374 0100 0474  ...ClassTest...t
 18 00000110: 6573 740c 000c 0010 0700 230c 0024 0025  est.......#..$.%
 19 00000120: 0c00 1100 1207 0026 0c00 2700 1001 0010  .......&..'.....
 20 00000130: 6a61 7661 2f6c 616e 672f 4f62 6a65 6374  java/lang/Object
 21 00000140: 0100 106a 6176 612f 6c61 6e67 2f53 7973  ...java/lang/Sys
 22 00000150: 7465 6d01 0003 6f75 7401 0015 4c6a 6176  tem...out...Ljav
 23 00000160: 612f 696f 2f50 7269 6e74 5374 7265 616d  a/io/PrintStream
 24 00000170: 3b01 0013 6a61 7661 2f69 6f2f 5072 696e  ;...java/io/Prin
 25 00000180: 7453 7472 6561 6d01 0007 7072 696e 746c  tStream...printl
```

使用`javap -verbose ClassTest`查看编译后的"Class"文件。

```
public class ClassTest
  minor version: 0
  major version: 55
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #3                          // ClassTest
  super_class: #9                         // java/lang/Object
  interfaces: 0, fields: 1, methods: 5, attributes: 1
Constant pool:
   #1 = Methodref          #9.#24         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#25         // ClassTest.name:Ljava/lang/String;
   #3 = Class              #26            // ClassTest
   #4 = String             #27            // test
   #5 = Methodref          #3.#28         // ClassTest."<init>":(Ljava/lang/String;)V
   #6 = Fieldref           #29.#30        // java/lang/System.out:Ljava/io/PrintStream;
   #7 = Methodref          #3.#31         // ClassTest.getName:()Ljava/lang/String;
   #8 = Methodref          #32.#33        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #9 = Class              #34            // java/lang/Object
  #10 = Utf8               name
  #11 = Utf8               Ljava/lang/String;
  #12 = Utf8               <init>
  #13 = Utf8               ()V
  #14 = Utf8               Code
  #15 = Utf8               LineNumberTable
  #16 = Utf8               (Ljava/lang/String;)V
  #17 = Utf8               getName
  #18 = Utf8               ()Ljava/lang/String;
  #19 = Utf8               setName
  #20 = Utf8               main
  #21 = Utf8               ([Ljava/lang/String;)V
  #22 = Utf8               SourceFile
  #23 = Utf8               ClassTest.java
  #24 = NameAndType        #12:#13        // "<init>":()V
  #25 = NameAndType        #10:#11        // name:Ljava/lang/String;
  #26 = Utf8               ClassTest
  #27 = Utf8               test
  #28 = NameAndType        #12:#16        // "<init>":(Ljava/lang/String;)V
  #29 = Class              #35            // java/lang/System
  #30 = NameAndType        #36:#37        // out:Ljava/io/PrintStream;
  #31 = NameAndType        #17:#18        // getName:()Ljava/lang/String;
  #32 = Class              #38            // java/io/PrintStream
  #33 = NameAndType        #39:#16        // println:(Ljava/lang/String;)V
  #34 = Utf8               java/lang/Object
  #35 = Utf8               java/lang/System
  #36 = Utf8               out
  #37 = Utf8               Ljava/io/PrintStream;
  #38 = Utf8               java/io/PrintStream
  #39 = Utf8               println
{
  public ClassTest();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 5: 0
        line 6: 4

  public ClassTest(java.lang.String);
    descriptor: (Ljava/lang/String;)V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: aload_1
         6: putfield      #2                  // Field name:Ljava/lang/String;
         9: return
      LineNumberTable:
        line 8: 0
        line 9: 4
        line 10: 9

  public java.lang.String getName();
    descriptor: ()Ljava/lang/String;
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field name:Ljava/lang/String;
         4: areturn
      LineNumberTable:
        line 13: 0

  public void setName(java.lang.String);
    descriptor: (Ljava/lang/String;)V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: putfield      #2                  // Field name:Ljava/lang/String;
         5: return
      LineNumberTable:
        line 17: 0
        line 18: 5

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #3                  // class ClassTest
         3: dup
         4: ldc           #4                  // String test
         6: invokespecial #5                  // Method "<init>":(Ljava/lang/String;)V
         9: astore_1
        10: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
        13: aload_1
        14: invokevirtual #7                  // Method getName:()Ljava/lang/String;
        17: invokevirtual #8                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        20: return
      LineNumberTable:
        line 21: 0
        line 22: 10
        line 23: 20
}

```

参考资料

* The Java® Virtual Machine Specification Java SE 11 Edition (jvms11)
* OpenJDK 11