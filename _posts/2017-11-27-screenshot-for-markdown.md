---
title: '自制Markdown截图利器'
layout: post
tags:
- Markdown
- Ubuntu
- Linux
comments: true
---

使用Markdown写作有不少好处，但是缺点之一就是插入截图的时候很麻烦。正常情况下，插入一张截图需要以下几个步骤：

- 截图并保存
- 上传图片至某个服务器
- 获取图片的链接地址并粘贴到Markdown

偶尔手动进行这样的操作可能还觉得没什么，但毕竟写技术博客时常需要用截图呈现效果，这么冗长的步骤就有些令人恼火了。而程序员毕竟是个“战术上懒惰，战略上勤奋”的物种，自然会想到自制工具，让电脑自动完成这些繁琐的步骤。正好最近准备开始系统学习Unix，于是手动撸了一个bash脚本：
```
#! /usr/bin/env bash

LOCAL_DIR=/media/dimitri/DATA/MyBlog/controny.github.io/assets/images/posts
REMOTE_DIR=https://controny.github.io/assets/images/posts
FILE_NAME=$(date +%Y%m%d%H%M%S).png
gnome-screenshot -a -f ${LOCAL_DIR}/${FILE_NAME}
echo \![]\(${REMOTE_DIR}/${FILE_NAME}\) | xclip -selection clipboard
```

**说明：**

- `LOCAL_DIR`：截图文件的本地保存目录。
- `REMOTE_DIR`：截图文件的远程目录，这里我直接用了我博客网站的目录，简单省事。
- `FILE_NAME`：图片的文件名，为了区分不同的截图，直接用精确到秒的当前时间（`$(date +%Y%m%d%H%M%S)`）作为文件名，这样就基本不会有命名的冲突了。
- 倒数第二行调用了Ubuntu自带的截图工具，其中`-a`表示区域（area）截图，即用鼠标拖动截图框；`-f`加上后面的路径指定截图文件的保存位置。
- 最后一行使用了一个叫做`xclip`的工具，用于将`echo`的输出结果（Markdown语法下的图片表示）复制到系统剪贴板。该工具可通过以下的命令安装：
```
sudo apt-get install xclip
```
如有兴趣，可查看具体的使用方法：
```
man xclip
```

然而，每次都要调用终端执行这个脚本也太不过瘾了，所以最后再设置一下快捷键。
打开系统设置中的键盘设置，再进入快捷键设置界面：
![](https://controny.github.io/assets/images/posts/20171127220557.png)
点击左下方的`+`，填写相关信息：
![](https://controny.github.io/assets/images/posts/20171127220747.png)
`Command`一栏我写的是
```
sh /home/dimitri/bin/screenshot_for_blog.sh
```
也就是执行脚本的指令。
点击保存后，再点击最右方设置快捷键，我设置的是`Ctrl`+`Alt`+`A`：
![](https://controny.github.io/assets/images/posts/20171127221122.png)
大功告成，现在插入截图的步骤就简化为：

- `Ctrl`+`Alt`+`A`
- 拖动鼠标
- `Ctrl`+`V`

嘿嘿，是不是非常方便呢？