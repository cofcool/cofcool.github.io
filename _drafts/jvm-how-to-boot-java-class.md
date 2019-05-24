# JVM 如何启动一个 java 程序

* JDK 11

stubRoutines

```cpp
// Calls to Java
  typedef void (*CallStub)(
    address   link,
    intptr_t* result,
    BasicType result_type,
    Method* method,
    address   entry_point,
    intptr_t* parameters,
    int       size_of_parameters,
    TRAPS
  );

static CallStub call_stub()                              { return CAST_TO_FN_PTR(CallStub, _call_stub_entry); }
```