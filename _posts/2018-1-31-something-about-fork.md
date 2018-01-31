---
layout: post
title:  "关于“fork函数返回两次”的理解"
tag:
- Unix
- 操作系统
comments: true
---

*Unix Network Programming*在介绍`fork`函数的时候这么写道：

>If you have never seen this function before, the hard part in understanding **fork** is that it is called *once* but it returns *twice*. 

这句话着实看得我云里雾里，一个函数被调用一次怎么会返回两次？？？

![](https://controny.github.io/assets/images/posts/20180131132135.png)

研究了一番才明白了这个函数的神奇之处。

![](https://controny.github.io/assets/images/posts/20180131132323.png)

首先讲讲**fork**这个单词的意思，这个的词最早的意思是叉子，但是程序员大概更多地是在下面这个地方看到这个词：

![](https://controny.github.io/assets/images/posts/20180131134352.png)

Github的**Fork**是指在项目的主分支上建立自己的分支，如果我们把这个图标放大，会发现它描绘得其实很形象：

![](https://controny.github.io/assets/images/posts/20180131134509.png)

而这个示意图同样适用于Unix中的`fork`：当父进程P调用`fork()`时，它将创建出一个与自身共享代码的子进程C（出现分叉点），随后P继续独立执行`fork()`以后的代码（左分支），然后C从`fork()`返回处开始执行与P共享的代码（右分支）。这样的话，虽然我们只写了一次`fork()`，但它实际上被返回了两次：先在P中返回，然后在C中返回。

接下来我们就利用一段测试代码实际体会一下：

```cpp
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("before forking\n----------\n");

    printf("after forking\n");
    pid_t pid = fork();
    if (pid == 0)
        printf("child process begins executing with pid: %d\n----------\n", getpid());
    else
        printf("parent process continues executing with pid: %d\n----------\n", getpid());

}

/*
before forking
----------
after forking
parent process continues executing with pid: 9043
----------
child process begins executing with pid: 9044
----------
*/

```

当`fork()`在父进程中返回时，返回值为子进程的pid（9044）；在子进程中返回时，返回值为0，所以我们可以根据这一点来区分父进程与子进程。另外，注意到子进程没有输出“before forking”、”after forking”等字样，这是因为，子进程是从`fork()`返回的地方（第8行）开始执行的。