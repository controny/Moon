---
layout: post
title:  "PyCharm下集成Google规范代码检查"
tag:
- 系统分析与设计
comments: true
---

项目后台决定选用Python开发，最开始原本计划以[Google风格](http://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_style_rules/)作为代码规范，后来发现直接用PyCharm默认集成的PEP8规范审查工具会省事许多，所以放弃了这个想法，但还是决定记录一下这折腾的过程。

回到正题，代码的规范化自然要引入代码静态检查工具的使用，在Python的领域中，pylint是一个佼佼者。

我们也推荐使用PyCharm这款强大的Python IDE进行开发，它不仅支持插件安装，还支持外部工具（如上述的pylint）的引入，极大地提高开发效率。下面就介绍怎样在PyCharm下利用pylint集成Google规范的代码检查。

### 安装pylint
用pip3（我们规定使用Python3开发）一键安装pylint
```python
pip3 install pylint
```

### 配置Google规范
因为pylint默认使用的是PEP8规范，与Google规范稍有出入，所以我们需要导入符合Google规范的配置文件。从[这里](https://gist.githubusercontent.com/rpunkfu/3cd6f8b8b23e27adedbd/raw/5138ece25240dde600a3ecf536f13efb243aa4ef/.pylintrc)下载配置文件保存到本地，运行下列命令即可得到指定文件的静态检查结果：
```python
pylint -rn --rcfile=<path to google pylintrc> <your.py>
```
其中，`--rcfile=<path to google pylintrc>`指定了上述配置文件的保存路径，`-rn`则设置pylint不输出最终的report，只输出检测到有问题的位置，以免我们被大量的表格信息所淹没。

### 集成到PyCharm中
目前为止，我们已经在命令行中成功使用了pylint，为了提高使用效率，下一步将其集成到PyCharm中。
首先依次点击`File`->`Settings`->`Tools`->`External Tools`进入外部工具管理页面：
![](https://controny.github.io/assets/images/posts/20180415120858.png)
再点击绿色+号添加新的工具：
![](https://controny.github.io/assets/images/posts/20180415121144.png)
分别填入

- Name: pylint。
- Program: pylint可执行文件所在位置，在终端中用`whereis pylint`查看然后复制上去。
- Arguments: 指运行pylint的参数，参考刚才的命令，我填的是`-rn --rcfile=~/.googlecl-pylintrc $FileName$`，其中，`~/.googlecl-pylintrc`是我保存的配置文件的路径，`$FileName$`代表当前的文件名。
- Working directory: `$FileDir$`，代表当前目录。

![](https://controny.github.io/assets/images/posts/20180415121527.png)
一路点击`OK`后，这个外部工具的设置就完成了。
在当前代码文件中随便点击一下，把焦点定位当前文件中，然后在上方导航栏中依次点击`Tools`->`External Tools`->`pylint`，即可用pylint检查当前代码：
![](https://controny.github.io/assets/images/posts/20180415143241.png)

