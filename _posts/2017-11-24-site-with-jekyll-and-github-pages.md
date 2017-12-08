---
title: '基于Github Pages和Jekyll的快速建站指南'
layout: post
tags:
- Jekyll
- Github Pages
comments: true
---

折腾了好几个小时，总算把基于Github Pages和Jekyll的博客网站建立起来了。回想了一下，整个过程其实很简单，但受限于教程的不够直观，我这个小白还是踩了不少坑，于是想总结一下，写一个快速建站指南。

# 概述
关于Github Pages和Jekyll，Jekyll的中文官方文档有如下的介绍

> Github Pages 是面向用户、组织和项目开放的公共静态页面搭建托管服 务，站点可以被免费托管在 Github 上，你可以选择使用 Github Pages 默 认提供的域名 github.io 或者自定义域名来发布站点。Github Pages 支持 自动利用 Jekyll 生成站点，也同样支持纯 HTML 文档，将你的 Jekyll 站 点托管在 Github Pages 上是一个不错的选择。

简而言之，你可以把Github Pages当做一个免费的Web服务器，官方有一个[极其简单的Hello World教程](https://pages.github.com/)，介绍了怎么把网页托管在Github上。
而Jekyll是一个自动生成网站的工具。为什么要用这个工具呢？首先要明确，我们的目标是快速搭建一个美观而实用的网站，因此对于技术上的细节考虑得越少越好。Jekyll就是帮助我们实现这一目标的利器，虽然只能生成静态网页，但对于一般的博客网站而言完全够用了。

---

# 快速建站
以上可能略显抽象，话不多说，下面我们就开启快速建站之旅吧～

- 挑选模板

要快速搭建一个有模有样的网站，套用别人开发好的模板自然是一个捷径。所以我们首先来[这个网站](http://themes.jekyllrc.org/)挑选一个自己喜欢的模板:
![](https://controny.github.io/assets/images/posts/Screenshot1.png)
随便点击一个模板，再点击`Demo`按钮就可以查看实际的运行效果：
![](https://controny.github.io/assets/images/posts/Screenshot2.png)

- 网站上线

选好模板后，就可以着手套用模板了，点击上图界面的`Homepage`按钮，进入该模板的Github仓库主页：
![](https://controny.github.io/assets/images/posts/201711241951.png)

**注意：下面假设你已经有一个Github账号且用户名为foo。**  
（意思就是你要把以下的所有`foo`改为你自己的用户名）  
点击右上角的`Fork`把该仓库Fork到自己的仓库，然后将仓库名改为`foo.github.io`：
![](https://controny.github.io/assets/images/posts/201711241955.png)
如果你有了上述Helloworld的尝试，这时候你可能会很兴奋地想打开`https://foo.github.io`看看实际效果，然而你实际看到的可能是：
![](https://controny.github.io/assets/images/posts/201711242001.png)
别灰心，这只是因为你还没有配置自己的`url`参数。打开根目录下的`_config.yml`文件，把`url`的值改为`https://foo.github.io`（有些模板可能要求你改为网站的子目录，所以，请仔细阅读注释）。  
现在再打开`https://foo.github.io`，你终于能看到你自己的网站上线了（尽管还只是个模板）！

- 本地编辑

接下来就是完善你自己的信息了，我们不需要写html、css、js，只需要编辑`_config.yml`或更换对应目录下的图片即可。不同的模板会有不同的说明，所以，还是那句话，仔细看文档。  
而在Github上编辑毕竟效率较低，因此我们一般会进行本地编辑。  
首先，在本地配置Jekyll的运行环境。以下是官方要求的基本环境：  
<https://jekyllrb.com/docs/installation/#requirements>  
如果都达到了要求的话，接下来安装Jekyll和Bundler：
```
sudo gem install jekyll bundler
```
如果一切正常的话，`git clone`自己的仓库到本地并进入对应的目录下，执行以下命令自动安装该模板的依赖文件：
```
sudo bundle install
```
最后执行
```
bundle exec jekyll serve
```
用浏览器打开`http://localhost:4000`就可以看到网站的运行效果了。  
本地调试满意后，push到Github上，你的个人网站就基本搭建完成了！
