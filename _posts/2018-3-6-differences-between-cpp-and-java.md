---
layout: post
title:  "Java与C++语法差异汇总"
tag:
- Java
- C++
comments: true
---

找实习时终于体会到了Java市场需求之大，于是重新拾起Java，做起了从C++过渡到Java的准备。意外在网上发现了一份Java与C++语法差异的[cheat sheet](https://www.cprogramming.com/tutorial/java/syntax-differences-java-c++.html)，整理如下。

## main函数
C++
```cpp
int main( int argc, char* argv[])
{
    printf( "Hello, world" );
}
```

在Java中，每个函数都必须在类中声明和定义，即使是main函数也不例外。
```java
class HelloWorld
{
    public static void main(String args[])
    {
        System.out.println( "Hello, World" );
    }
}
```

## 类的声明
C++
```cpp
class Bar {
    Bar();
    ~Bar();
};
```

Java类方法的定义也必须写在类里，没有C++那种析构函数。提供静态初始化块（static initialization block）初始化类静态变量，而C++的类静态变量不能在类中初始化。此外，类的声明最后不需要分号。
```java
class Bar {
    static private int x;
    // static initialization block
    { x = 5; }

    Bar() {}
}
```

## 类静态方法访问
C++在类名后用域解析符`::`访问。
```cpp
class MyClass
{
    public:
    static doStuff();
};

MyClass::doStuff();
```

Java直接用`.`访问，就和对象访问非静态方法的语法一样。
```java
class MyClass
{
    public static doStuff()
    {
        // do stuff
    }
}

MyClass.doStuff();
```

## 对象声明
C++允许在栈中声明（静态声明）或在堆中声明（动态声明）。
```cpp
// on the stack
myClass x;
// or on the heap
myClass *x = new myClass();
delete x;
```

Java只允许在堆中声明，而由于Java的垃圾自动回收机制，以`new`的方式构造的对象不必显式地销毁。
```java
myClass x = new myClass();
```

## 继承
C++
```cpp
class Foo : public Bar
    { ... };
```

Java
```java
class Foo extends Bar
    { ... }
```

## 虚函数
C++用`virtual`关键字声明虚函数。

Java默认采用虚函数，可用`final`修饰禁止函数重写（overriding)，所以下面的代码会发生运行时错误：
```java
// Base.java
public class Base {
    final void foo() {
        System.out.println("Base");
    }
}

// Derived.java
public class Derived extends Base {
    void foo() {
        System.out.println("Derived");
    }

    public static void main(String[] args) {
        Base d1 = new Derived();
        d1.foo();
        Derived d2 = new Derived();
        d2.foo();
    }
}
```

## 常量声明
C++用`const`声明常量。

Java用`final`声明常量。但`final`的作用还包括禁止子类重写函数（如上一条）、禁止类被继承。

## 数组声明
与对象的声明类似，C++的数组也有两种声明方式。
```cpp
int x[10];
// or 
int *x = new x[10];
// use x, then reclaim memory
delete[] x;
```

Java只有动态声明的方式，同时因为Java不存在指针的概念，注意以`int[]`的形式取代`int*`。
```java
int[] x = new int[10];
```

