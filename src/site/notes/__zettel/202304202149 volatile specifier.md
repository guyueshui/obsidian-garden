---
{"dg-publish":true,"permalink":"/zettel/202304202149-volatile-specifier/","title":202304202149,"tags":["cpp","c++","specifier","volatile"],"created":"2023-04-20T21:49:12+08:00"}
---


volatile修饰的变量旨在告诉编译器这个变量易失（对应缓存的术语）、易变，因此在读取变量值的时候，始终通过内存取址的方式，而不信赖缓存。

例如：

```cpp
int i = 10;
int a = i;
// 以编译器不知道的方式改变i的值为32
int b = i;
cout << b;  // 10
```

这里，由于编译器的优化，编译器看到两次赋值间，i的值未发生变化。所以直接从缓存（寄存器）中拿出上次的值赋给b.

```cpp
volatile int i = 10;
int a = i;
// 以编译器不知道的方式改变i的值为32
int b = i;
cout << b;  // 32
```

这里，用volatile修饰变量i，告诉编译器这个变量易失，不要相信缓存，不要从寄存器读取。所以编译器按照常规方式从内存地址读取i的值（32）赋给b.

see also