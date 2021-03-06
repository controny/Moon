---
layout: post
title: "shell中的单引号与双引号"
tags:
- shell
- Linux
comments: true
---

在Python、JS等语言中，字符串可以用单引号或双引号表示，两者是等价的，而在shell中，单引号和双引号有着细微但致命的区别，因此需要特别注意。

# 强引用与弱引用

* 单引号属于**强引用**，即单引号中的所有字符都没有特殊含义。
* 双引号属于**弱引用**，$(美元)、\`(反引号)、\\(反斜线)这三个字符在双引号中保留其原来的特殊含义，其余字符均不保留。

# 例子

举个例子，以下的命令只会把单引号里的内容原封不动地输出出来，即`I am $USER`。
```
echo 'I am $USER'
```
我的`USER`变量是`dimitri`，如果希望得到`I am dimitri`这样的输出，按照上述规则，就需要使用双引号：
```
echo "I am $USER"
```