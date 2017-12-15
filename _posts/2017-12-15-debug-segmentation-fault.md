---
layout: post
title:  "简单调试Segmentation Fault"
tag:
- C
comments: true
---

最近在写编译器设计的project作业时，一直用的是以C语言为基础的yacc，自然，也不得不再次遭遇C语言的许多大坑。其中最甚者便是Segmentation Fault。

# 什么是Segmentation Fault
[维基百科](https://en.wikipedia.org/wiki/Segmentation_fault)对此有简单明了的定义：

>In computing, a **segmentation fault** (often shortened to segfault) or access violation is a fault, or failure condition, raised by hardware with memory protection, notifying an operating system (OS) the software has attempted to access a restricted area of memory (a memory access violation).

简单地用中文说，就是访问了不属于该进程的内存。而我总是对一个名词的来源感兴趣，为什么这一现象要称为Segmentation Fault呢？  
起初，我一直以为是与早期操作系统的分段式内存管理机制有关（memory segmentation ），在网上查了一下才发现两者并无直接联系：

>"Segmentation" isn't at all related to the old "segmented memory model" used by early x86 processors; it's an earlier use which just refers to a portion or *segment* of memory.

原来segmentation仅仅是指一段（segment）内存o(╯□╰)o

# 什么情况会产生Segmentation Fault
在写C语言的时候，最容易产生Segmentation Fault的大概有两种情况：

- 访问空指针指向的内存
- 越界访问数组

第一种情况最简单的示例就是：
```
int main(int argc, char* argv[])
{
    int *p = NULL;
    *p = 1;
	
    return 0;
}
```
这里，空指针`p`被设为NULL，指向的内存便不属于该进程，对`p`指向的内存赋值自然就属于非法访问内存了。  
第二种情况大概是每个学过C语言的人都会遇到的。然而，越界访问并不会必然导致Segmentation Fault，因为有时候你会“幸运”地访问到同属于该进程的一块内存。当然，这种“幸运”其实是一种巨大的不幸，出现这种情况时，运行时环境不会直接了当地告诉你有错误，而是会以莫名其妙地方式暗示你。

# 如何调试Segmentation Fault
Segmentation Fault令人头疼的地方就是，运行时环境并不会直接告诉你代码的哪一行出现了问题。过去，我的做法一直是凭猜测，然后在可能的地方输出特定信息以观察程序是否在该位置出错。直到我意识到可以用强大的调试工具GDB来提高效率。  
如第一种情况的示例，在编译的时候加入`-g`参数可令其生成调试信息以便待会用GDB调试：
```
gcc -g c_test.c
```
然后就是用GDB执行生成可执行文件：
```
gdb ./a.out
```
进入交互界面后输入`r`让程序开跑，之后就能看到非常醒目的错误信息：
```
Program received signal SIGSEGV, Segmentation fault.
0x00000000004004ed in main (argc=1, argv=0x7fffffffdd38) at c_test.c:7
7	    *p = 1;
```
于是便瞬间锁定了错误所在的位置，又可以愉快改下一个bug了～