# 关于 Jva 8 字符串变量引用问题的讨论

## JLS

### 3.10.5 String Literals


A string literal consists of zero or more characters enclosed in double quotes. Characters may be represented by escape sequences (§3.10.6) - one escape sequence for characters in the range U+0000 to U+FFFF, two escape sequences for the UTF-16 surrogate code units of characters in the range U+010000 to U +10FFFF.
35
3.10 Literals
LEXICAL STRUCTURE
36
StringLiteral:
" {StringCharacter} "
StringCharacter: InputCharacter but not " or \ EscapeSequence
See §3.10.6 for the definition of EscapeSequence.
A string literal is always of type String (§4.3.3).
It is a compile-time error for a line terminator to appear after the opening " and before the closing matching ".
As specified in §3.4, the characters CR and LF are never an InputCharacter; each is recognized as constituting a LineTerminator.
A long string literal can always be broken up into shorter pieces and written as a (possibly parenthesized) expression using the string concatenation operator + (§15.18.1).
The following are examples of string literals:
""
"\""
"This is a string"
"This is a " +
    "two-line string"
// the empty string
// a string containing " alone
// a string containing 16 characters
// actually a string-valued constant expression,
   // formed from two string literals
Because Unicode escapes are processed very early, it is not correct to write "\u000a" for a string literal containing a single linefeed (LF); the Unicode escape \u000a is transformed into an actual linefeed in translation step 1 (§3.3) and the linefeed becomes a LineTerminator in step 2 (§3.4), and so the string literal is not valid in step 3. Instead, one should write "\n" (§3.10.6). Similarly, it is not correct to write "\u000d" for a string literal containing a single carriage return (CR). Instead, use "\r". Finally, it is not possible to write "\u0022" for a string literal containing a double quotation mark (").
A string literal is a reference to an instance of class String (§4.3.1, §4.3.3).
Moreover, a string literal always refers to the same instance of class String. This is because string literals - or, more generally, strings that are the values of constant expressions (§15.28) - are "interned" so as to share unique instances, using the method String.intern.
Example 3.10.5-1. String Literals
The program consisting of the compilation unit (§7.3):
package testPackage;

LEXICAL STRUCTURE
Literals 3.10
       class Test {
           public static void main(String[] args) {
               String hello = "Hello", lo = "lo";
               System.out.print((hello == "Hello") + " ");
               System.out.print((Other.hello == hello) + " ");
               System.out.print((other.Other.hello == hello) + " ");
               System.out.print((hello == ("Hel"+"lo")) + " ");
               System.out.print((hello == ("Hel"+lo)) + " ");
               System.out.println(hello == ("Hel"+lo).intern());
} }
       class Other { static String hello = "Hello"; }
and the compilation unit:
       package other;
       public class Other { public static String hello = "Hello"; }
produces the output:
       true true true true false true
This example illustrates six points:
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

## JVMS

### 5.1 The Run-Time Constant Pool

A string literal is a reference to an instance of class String, and is derived from a CONSTANT_String_info structure (§4.4.3) in the binary representation of a class or interface. The CONSTANT_String_info structure gives the sequence of Unicode code points constituting the string literal.
The Java programming language requires that identical string literals (that is, literals that contain the same sequence of code points) must refer to the same instance of class String (JLS §3.10.5). In addition, if the method String.intern is called on any string, the result is a reference to the same class instance that would be returned if that string appeared as a literal. Thus, the following expression must have the value true:
     ("a" + "b" + "c").intern() == "abc"
To derive a string literal, the Java Virtual Machine examines the sequence of code points given by the CONSTANT_String_info structure.
– If the method String.intern has previously been called on an instance of class String containing a sequence of Unicode code points identical to that given by the CONSTANT_String_info structure, then the result of string literal derivation is a reference to that same instance of class String.
– Otherwise, a new instance of class String is created containing the sequence of Unicode code points given by the CONSTANT_String_info structure; a reference to that class instance is the result of string literal derivation. Finally, the intern method of the new String instance is invoked.

* The Java® Language Specification Java SE 8 Edition
* The Java® Virtual Machine Specification Java SE 8 Edition
