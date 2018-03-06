---
layout: post
title:  "浅谈Unix五大I/O模型"
tag:
- Unix
- 操作系统
comments: true
---

## 5种I/O模型

Unix支持5种I/O模型，下面就一一介绍一下。

### 阻塞式I/O (Blocking I/O)

在阻塞式I/O中，应用进程从“调用某I/O函数”开始到“该函数返回”的整段时间都是被阻塞的。

如下的例子中，应用进程就阻塞于I/O函数`recvfrom`的调用：
![](https://controny.github.io/assets/images/posts/20180306173637.png)

### 非阻塞式I/O (Nonblocking I/O)

在非阻塞式I/O中，当应用进程调用I/O函数时，若所需的数据未准备好，函数直接返回一个错误而不阻塞进程。

应用该模型时，进程往往会循环调用I/O函数以查看数据是否就绪，这样的机制又被称为轮询（polling），比如下面的例子：
![](https://controny.github.io/assets/images/posts/20180306174424.png)

### I/O复用 (I/O multiplexing)

需要调用`select`、`poll`或`epoll`机制充当“代理人”的角色，也有阻塞产生，但与阻塞式I/O不同的是，阻塞发生在“代理人”身上，而不是I/O函数上。

下面是以`select`机制实现的例子：
![](https://controny.github.io/assets/images/posts/20180306190658.png)

### 信号驱动式I/O (Signal-driven I/O)

首先设定好信号处理程序，当数据就绪时才发送信号令I/O函数处理数据：
![](https://controny.github.io/assets/images/posts/20180306191132.png)

### 异步I/O (Asynchronous I/O)

与信号驱动式I/O很像，主要区别在于，信号驱动式I/O告知的是何时可以开始I/O操作（如把数据从内核缓冲区复制到用户缓冲区），而异步I/O告知的是I/O操作何时完成：
![](https://controny.github.io/assets/images/posts/20180306191629.png)

## 比较

![](https://controny.github.io/assets/images/posts/20180306191726.png)

同步I/O操作和异步I/O操作的定义分别是：

- 同步I/O操作导致请求进程阻塞，直到I/O操作完成。
- 异步I/O操作不导致请求进程阻塞。

因此以上只有异步I/O模型属于**异步I/O操作**，其余均属于**同步I/O操作**。这是因为，前四种模型中，尽管等待的时候不一定都是阻塞的，但真正的I/O操作（把数据从内核缓冲区复制到用户缓冲区）却都会阻塞进程。

