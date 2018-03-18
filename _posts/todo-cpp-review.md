---
layout: post
title:  "C++相关问题"
tag:
- C++
comments: true
---

### extern与static的作用

- extern可以置于变量或者函数前，以标示变量或者函数的定义在别的文件中，提示编译器遇到此变量和函数时在其他模块中寻找其定义。另外，`extern "C"`语句实现了类C（类C代表的是跟C语言的编译和链接方式一致的所有语言）和C++的混合编程：在C++源文件中的语句前面加上extern "C"，表明它按照类C的编译和链接规约来编译和链接，而不是C++的编译和链接规约，这样在类C的代码中就可以调用C++的函数或变量等，更多详情参考<http://www.cnblogs.com/skynet/archive/2010/07/10/1774964.html>。
- static的作用则与extern“水火不相容”，它限制变量或函数只能在本文件中被访问。

```cpp
// test1.cpp
int a = 1;
int b = 2;
int c = a;

// test2.cpp
#include <iostream>

extern int a;
// int b = 4;
// error: multiple definition of `b`
static int c = 5;

int main(void) {
    std::cout << "a == " << a << std::endl;
    // a == 1
    std::cout << "c == " << c << std::endl;
    // c == 5
    return 0;
}
```
上述代码中，
- 以`extern`声明的变量`a`相当于是两个文件的“全局变量”。
- `b`没有以`extern`声明，却同时被两个文件都声明了，所以链接时会产生重复定义的错误。
- `c`以`static`声明，只能在本文件内访问，两个文件各自有一个独立的`c`，所以不受影响。

### new operator、operator new与placement new
- new operator就是一般所说的**new**，用法如`A* pa = new A()`，它是C++操作符，是关键字。它的内部操作有三步：调用下面要讲的operator new分配空间，然后调用相关对象的初始化函数，最后返回指针。
- operator new与malloc作用类似，可看成是一个用于分配内存而不构造对象的函数，如`A* ptr = (A*)operator new(sizeof(A))`。operator new既然可看成一个函数，所以也可以被重载——一般在强调内存分配效率的情况下会考虑重载operator new。
- 在上一项中，我用operator new分配了一块内存，然后就遇到了一个问题：怎么在这块内存上面构造对象呢？这就要用到placement new了，它实际上是operator new的一个重载版本，原型是`void* operator new (std::size_t size, void* ptr) noexcept;`，这里的`size`参数是被忽略的，只需传入已分配内存的指针`ptr`参数，目的是在已分配的内存上构造对象，最后将原指针返回。它的用法也与普通函数不同，需在函数后提供构造函数，如，我们可以这样解决上述问题：`A* pa = new(ptr) A()`。

### new和malloc的区别
- malloc是库函数，new是C++关键字。
- new自行计算空间大小，如`int* a = new int[10]`；而malloc需要指定空间大小，如`int* a = malloc( sizeof(int)*10 )`。
- new在动态分配空间时会可调用对象的构造函数初始化对象；而malloc只是分配一段给定大小的内存。
- new分配失败时会抛出异常；而malloc只会返回NULL。
- new可以通过调用malloc来实现，反之则不可以。

### C++多态性
多态性可以简单理解为“一个接口，多种方法”，程序在运行时才决定调用哪一种方法。C++的多态性是通过虚函数来实现的，虚函数允许子类重新定义父类的方法，即**重写（override）**。它与**重载（overload）**的区别是：重写主要针对名称相同、参数列表相同的函数，运行时才决定函数地址；而重载针对名称相同、参数列表不同的函数，编译期即可决定调用哪个函数，不体现多态性。

### 虚函数的实现
虚函数是通过**vtable（virtual table，虚函数表）**实现的。
注意该表是**针对类的**，而不是针对对象的，也就是说，编译器会为每一个有虚函数的类维护一张vtable，该类的每一个实例都会包含一个表指针，指向同一张表，如下图所示：
![](https://controny.github.io/assets/images/posts/20180312201231.png)
C1和C2的vtable都分别只创建了一张，它们的实例通过表指针访问对应的vtable。  
当一个子类重写了虚函数，其vtable的相关条目也会被替换为合适的函数地址。比如，假设上述的C2是C1的子类，C2重写了一些虚函数，也增加了一个虚函数，最终两者的vtable就分别是：
![](https://controny.github.io/assets/images/posts/20180312201903.png)
![](https://controny.github.io/assets/images/posts/20180312201913.png)

下面再通过一段测试代码实际感受一下。
```cpp
#include <iostream>

using namespace std;

class Base {
public:
    virtual void f() { cout << "Base::f" << endl; }
    virtual void g() { cout << "Base::g" << endl; }
    virtual void h() { cout << "Base::h" << endl; }
};

class Derived: public Base {
public:
    virtual void f() { cout << "Derived::f" << endl; }
    virtual void g1() { cout << "Derived::g1" << endl; }
    virtual void h1() { cout << "Derived::h1" << endl; }
};

int main(void) {
    Base* b = new Base();
    Base* d1 = new Derived();
    Derived* d2 = new Derived();
    Derived* d3 = new Derived();

    return 0;
}
```
我们可以利用GDB的`info vtbl [obj]`命令来查看对象所指的vtable：
![](https://controny.github.io/assets/images/posts/20180312202024.png)
可以看到：
- 作为`Base`类的对象`b`所指的vtable包含了它三个虚函数的地址。
- `d1`是`Derived`类，对应的vtable中，`f()`是`Derived`的重写版本，体现了多态性。而因为它由`Base`指针指向，所以只能访问vtable中的父类函数。
- 注意到`d1`、`d2`、`d3`对应的vtable地址是一致的，证明它们都是同一张表。

### 引用与指针的区别
- 指针是个实体，有自己的内存空间；引用只是一块内存的别名，编译器没有为其分配空间。
- 指针可以不初始化，可为空；引用必须初始化，不可为空。
- 指针可以改变其指向的对象；引用则不可变。
- 实际上，引用可以做的事情指针都可以完成，但引用更加安全。
