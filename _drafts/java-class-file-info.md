# 简述 Java Class 文件

jvms11 文档中的 4. The class File Format

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

sun.jvm.hotspot.runtime.ClassConstants
sun.jvm.hotspot.oops.Klass
com.sun.tools.javac.jvm.ClassFile

```java
public final static int JAVA_MAGIC = 0xCAFEBABE;
```

参考资料

* The Java® Virtual Machine Specification Java SE 11 Edition (jvms11)
* OpenJDK 11