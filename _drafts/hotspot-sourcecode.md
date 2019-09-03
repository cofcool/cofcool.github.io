jdk11-1ddf9a99e4ad

`oopDesc`, 对象基类定义, 使用"C++"描述"Java"对象

oop.hpp
```cpp
class oopDesc {
 private:
  volatile markOop _mark;
  union _metadata {
    Klass*      _klass;
    narrowKlass _compressed_klass;
  } _metadata;
  ...
}
```

`Klass`, 语言层面的类对象; 对象在"vm"层面的行为分发
klass.hpp
```cpp
// One reason for the oop/klass dichotomy in the implementation is
// that we don't want a C++ vtbl pointer in every object.  Thus,
// normal oops don't have any virtual functions.  Instead, they
// forward all "virtual" functions to their klass, which does have
// a vtbl and does the C++ dispatch depending on the object's
// actual type.  (See oop.inline.hpp for some of the forwarding code.)
// ALL FUNCTIONS IMPLEMENTING THIS DISPATCH ARE PREFIXED WITH "oop_"!
class Klass : public Metadata {
  jint        _layout_helper;
  const KlassID _id;
  juint       _super_check_offset;
  Symbol*     _name;
  ...
  OopHandle _java_mirror;
  Klass*      _super;
  Klass*      _subklass;
  ClassLoaderData* _class_loader_data;
  jint        _modifier_flags;
  AccessFlags _access_flags;
  ...
  jlong    _last_biased_lock_bulk_revocation_time;
  markOop  _prototype_header;
  jint     _biased_lock_revocation_count;
}
```