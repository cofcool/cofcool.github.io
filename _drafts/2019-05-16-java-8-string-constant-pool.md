# 关于 Java 8 字符串变量引用问题的讨论


`JLS8` 的 `3.10.5 String Literals`中对字符串字面量的定义：

> A string literal consists of zero or more characters enclosed in double quotes. Characters may be represented by escape sequences (§3.10.6) - one escape sequence for characters in the range U+0000 to U+FFFF, two escape sequences for the UTF-16 surrogate code units of characters in the range U+010000 to U +10FFFF.


一个`String`类型的变量常会指向同一个实例引用。这是因为它可通过`String.intern`方法转为`interned`变量，并指向"字符串常量池"中的实例引用。

```java
public native String intern();
```

Moreover, a string literal always refers to the same instance of class String. This is because string literals - or, more generally, strings that are the values of constant expressions (§15.28) - are "interned" so as to share unique instances, using the method String.intern.

如下例子：

```java
class Test {
       public static void main(String[] args) {
               String hello = "Hello", lo = "lo";
               System.out.print((hello == "Hello") + " ");
               System.out.print((Other.hello == hello) + " ");
               System.out.print((other.Other.hello == hello) + " ");
               System.out.print((hello == ("Hel"+"lo")) + " ");
               System.out.print((hello == ("Hel"+lo)) + " ");
               System.out.println(hello == ("Hel"+lo).intern());
       } 
}
class Other {
       static String hello = "Hello"; 
}
```

other 包下的 Other 类:

```java
package other;
public class Other { 
       public static String hello = "Hello";
 }
       
```

结果:
```
true true true true false true
```

经过 "Oracle JDK 1.8" 编译后的字节码如下：
```
class Test
  minor version: 0
  major version: 52
  flags: (0x0020) ACC_SUPER
  this_class: #16                         // Test
  super_class: #17                        // java/lang/Object
  interfaces: 0, fields: 0, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #17.#39        // java/lang/Object."<init>":()V
   #2 = String             #40            // Hello
   #3 = String             #31            // lo
   #4 = Fieldref           #41.#42        // java/lang/System.out:Ljava/io/PrintStream;
   #5 = Class              #43            // java/lang/StringBuilder
   #6 = Methodref          #5.#39         // java/lang/StringBuilder."<init>":()V
   #7 = Methodref          #5.#44         // java/lang/StringBuilder.append:(Z)Ljava/lang/StringBuilder;
   #8 = String             #45            //
   #9 = Methodref          #5.#46         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #10 = Methodref          #5.#47         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #11 = Methodref          #48.#49        // java/io/PrintStream.print:(Ljava/lang/String;)V
  #12 = Fieldref           #50.#51        // Other.hello:Ljava/lang/String;
  #13 = String             #52            // Hel
  #14 = Methodref          #53.#54        // java/lang/String.intern:()Ljava/lang/String;
  #15 = Methodref          #48.#55        // java/io/PrintStream.println:(Z)V
  #16 = Class              #56            // Test
  #17 = Class              #57            // java/lang/Object
  #18 = Utf8               <init>
  #19 = Utf8               ()V
  #20 = Utf8               Code
  #21 = Utf8               LineNumberTable
  #22 = Utf8               LocalVariableTable
  #23 = Utf8               this
  #24 = Utf8               LTest;
  #25 = Utf8               main
  #26 = Utf8               ([Ljava/lang/String;)V
  #27 = Utf8               args
  #28 = Utf8               [Ljava/lang/String;
  #29 = Utf8               hello
  #30 = Utf8               Ljava/lang/String;
  #31 = Utf8               lo
  #32 = Utf8               StackMapTable
  #33 = Class              #28            // "[Ljava/lang/String;"
  #34 = Class              #58            // java/lang/String
  #35 = Class              #59            // java/io/PrintStream
  #36 = Class              #43            // java/lang/StringBuilder
  #37 = Utf8               SourceFile
  #38 = Utf8               Test.java
  #39 = NameAndType        #18:#19        // "<init>":()V
  #40 = Utf8               Hello
  #41 = Class              #60            // java/lang/System
  #42 = NameAndType        #61:#62        // out:Ljava/io/PrintStream;
  #43 = Utf8               java/lang/StringBuilder
  #44 = NameAndType        #63:#64        // append:(Z)Ljava/lang/StringBuilder;
  #45 = Utf8
  #46 = NameAndType        #63:#65        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #47 = NameAndType        #66:#67        // toString:()Ljava/lang/String;
  #48 = Class              #59            // java/io/PrintStream
  #49 = NameAndType        #68:#69        // print:(Ljava/lang/String;)V
  #50 = Class              #70            // Other
  #51 = NameAndType        #29:#30        // hello:Ljava/lang/String;
  #52 = Utf8               Hel
  #53 = Class              #58            // java/lang/String
  #54 = NameAndType        #71:#67        // intern:()Ljava/lang/String;
  #55 = NameAndType        #72:#73        // println:(Z)V
  #56 = Utf8               Test
  #57 = Utf8               java/lang/Object
  #58 = Utf8               java/lang/String
  #59 = Utf8               java/io/PrintStream
  #60 = Utf8               java/lang/System
  #61 = Utf8               out
  #62 = Utf8               Ljava/io/PrintStream;
  #63 = Utf8               append
  #64 = Utf8               (Z)Ljava/lang/StringBuilder;
  #65 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #66 = Utf8               toString
  #67 = Utf8               ()Ljava/lang/String;
  #68 = Utf8               print
  #69 = Utf8               (Ljava/lang/String;)V
  #70 = Utf8               Other
  #71 = Utf8               intern
  #72 = Utf8               println
  #73 = Utf8               (Z)V
{
  Test();
    descriptor: ()V
    flags: (0x0000)
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   LTest;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=5, locals=3, args_size=1
         0: ldc           #2                  // String Hello
         2: astore_1
         3: ldc           #3                  // String lo
         5: astore_2
         6: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
         9: new           #5                  // class java/lang/StringBuilder
        12: dup
        13: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
        16: aload_1
        17: ldc           #2                  // String Hello
        19: if_acmpne     26
        22: iconst_1
        23: goto          27
        26: iconst_0
        27: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Z)Ljava/lang/StringBuilder;
        30: ldc           #8                  // String
        32: invokevirtual #9                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        35: invokevirtual #10                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        38: invokevirtual #11                 // Method java/io/PrintStream.print:(Ljava/lang/String;)V
        41: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
        44: new           #5                  // class java/lang/StringBuilder
        47: dup
        48: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
        51: getstatic     #12                 // Field Other.hello:Ljava/lang/String;
        54: aload_1
        55: if_acmpne     62
        58: iconst_1
        59: goto          63
        62: iconst_0
        63: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Z)Ljava/lang/StringBuilder;
        66: ldc           #8                  // String
        68: invokevirtual #9                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        71: invokevirtual #10                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        74: invokevirtual #11                 // Method java/io/PrintStream.print:(Ljava/lang/String;)V
        77: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
        80: new           #5                  // class java/lang/StringBuilder
        83: dup
        84: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
        87: getstatic     #12                 // Field Other.hello:Ljava/lang/String;
        90: aload_1
        91: if_acmpne     98
        94: iconst_1
        95: goto          99
        98: iconst_0
        99: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Z)Ljava/lang/StringBuilder;
       102: ldc           #8                  // String
       104: invokevirtual #9                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
       107: invokevirtual #10                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
       110: invokevirtual #11                 // Method java/io/PrintStream.print:(Ljava/lang/String;)V
       113: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
       116: new           #5                  // class java/lang/StringBuilder
       119: dup
       120: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
       123: aload_1
       124: ldc           #2                  // String Hello
       126: if_acmpne     133
       129: iconst_1
       130: goto          134
       133: iconst_0
       134: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Z)Ljava/lang/StringBuilder;
       137: ldc           #8                  // String
       139: invokevirtual #9                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
       142: invokevirtual #10                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
       145: invokevirtual #11                 // Method java/io/PrintStream.print:(Ljava/lang/String;)V
       148: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
       151: new           #5                  // class java/lang/StringBuilder
       154: dup
       155: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
       158: aload_1
       159: new           #5                  // class java/lang/StringBuilder
       162: dup
       163: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
       166: ldc           #13                 // String Hel
       168: invokevirtual #9                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
       171: aload_2
       172: invokevirtual #9                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
       175: invokevirtual #10                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
       178: if_acmpne     185
       181: iconst_1
       182: goto          186
       185: iconst_0
       186: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Z)Ljava/lang/StringBuilder;
       189: ldc           #8                  // String
       191: invokevirtual #9                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
       194: invokevirtual #10                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
       197: invokevirtual #11                 // Method java/io/PrintStream.print:(Ljava/lang/String;)V
       200: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
       203: aload_1
       204: new           #5                  // class java/lang/StringBuilder
       207: dup
       208: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
       211: ldc           #13                 // String Hel
       213: invokevirtual #9                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
       216: aload_2
       217: invokevirtual #9                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
       220: invokevirtual #10                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
       223: invokevirtual #14                 // Method java/lang/String.intern:()Ljava/lang/String;
       226: if_acmpne     233
       229: iconst_1
       230: goto          234
       233: iconst_0
       234: invokevirtual #15                 // Method java/io/PrintStream.println:(Z)V
       237: return
      LineNumberTable:
        line 3: 0
        line 4: 6
        line 5: 41
        line 6: 77
        line 7: 113
        line 8: 148
        line 9: 200
        line 10: 237
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0     238     0  args   [Ljava/lang/String;
            3     235     1 hello   Ljava/lang/String;
            6     232     2    lo   Ljava/lang/String;
      StackMapTable: number_of_entries = 12
        frame_type = 255 /* full_frame */
          offset_delta = 26
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, class java/lang/StringBuilder ]
        frame_type = 255 /* full_frame */
          offset_delta = 0
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, class java/lang/StringBuilder, int ]
        frame_type = 255 /* full_frame */
          offset_delta = 34
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, class java/lang/StringBuilder ]
        frame_type = 255 /* full_frame */
          offset_delta = 0
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, class java/lang/StringBuilder, int ]
        frame_type = 255 /* full_frame */
          offset_delta = 34
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, class java/lang/StringBuilder ]
        frame_type = 255 /* full_frame */
          offset_delta = 0
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, class java/lang/StringBuilder, int ]
        frame_type = 255 /* full_frame */
          offset_delta = 33
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, class java/lang/StringBuilder ]
        frame_type = 255 /* full_frame */
          offset_delta = 0
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, class java/lang/StringBuilder, int ]
        frame_type = 255 /* full_frame */
          offset_delta = 50
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, class java/lang/StringBuilder ]
        frame_type = 255 /* full_frame */
          offset_delta = 0
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, class java/lang/StringBuilder, int ]
        frame_type = 110 /* same_locals_1_stack_item */
          stack = [ class java/io/PrintStream ]
        frame_type = 255 /* full_frame */
          offset_delta = 0
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, int ]
}
SourceFile: "Test.java"

```

这个例子揭示了如下六点:

* 同一包下的同一类的字符串字面量指向同一个`String`对象
* 同一包下的不同类的字符串字面量指向同一个`String`对象
* 不同包下的不同类的字符串字面量指向同一个`String`对象
* 编译时通过计算而创建的字符串字面量是相同对象
* 运行时通过计算而创建的字符串字面量是不同对象
* 调用`String.intern`返回的字符串字面量和已存在并内容一致的变量是同一对象

• Literal strings within the same class (§8 (Classes)) in the same package (§7 (Packages))
represent references to the same String object (§4.3.1).
• Literal strings within different classes in the same package represent references to the
same String object.
• Literal strings within different classes in different packages likewise represent references
to the same String object.
• Strings computed by constant expressions (§15.28) are computed at compile time and
then treated as if they were literals.
• Strings computed by concatenation at run time are newly created and therefore distinct.
• The result of explicitly interning a computed string is the same string as any pre-existing literal string with the same contents.


`JVMS`的`5.1 运行时常量池`一文介绍了字符串字面量常量池。

> A string literal is a reference to an instance of class String, and is derived from a CONSTANT_String_info structure (§4.4.3) in the binary representation of a class or interface. The CONSTANT_String_info structure gives the sequence of Unicode code points constituting the string literal.

> The Java programming language requires that identical string literals (that is, literals that contain the same sequence of code points) must refer to the same instance of class String (JLS §3.10.5). In addition, if the method String.intern is called on any string, the result is a reference to the same class instance that would be returned if that string appeared as a literal. Thus, the following expression must have the value true:
     ("a" + "b" + "c").intern() == "abc"
To derive a string literal, the Java Virtual Machine examines the sequence of code points given by the CONSTANT_String_info structure.

> If the method String.intern has previously been called on an instance of class String containing a sequence of Unicode code points identical to that given by the CONSTANT_String_info structure, then the result of string literal derivation is a reference to that same instance of class String.

> Otherwise, a new instance of class String is created containing the sequence of Unicode code points given by the CONSTANT_String_info structure; a reference to that class instance is the result of string literal derivation. Finally, the intern method of the new String instance is invoked.

## 参考资料

* The Java® Language Specification Java SE 8 Edition
* The Java® Virtual Machine Specification Java SE 8 Edition
