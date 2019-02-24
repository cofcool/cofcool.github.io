# 关于 Java 8 字符串变量引用问题的讨论


`JLS8` 的 `3.10.5 String Literals`中对字符串字面量的定义：

> A string literal consists of zero or more characters enclosed in double quotes. Characters may be represented by escape sequences (§3.10.6) - one escape sequence for characters in the range U+0000 to U+FFFF, two escape sequences for the UTF-16 surrogate code units of characters in the range U+010000 to U +10FFFF.

Java

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
