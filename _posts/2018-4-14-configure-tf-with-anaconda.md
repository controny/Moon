---
layout: post
title:  "利用Anaconda配置TensorFlow虚拟运行环境"
tag:
- Anaconda
- TensorFlow
comments: true
---

最近要开始在集群上跑深度学习实验了，发现默认配置的TensorFlow版本太旧，工欲善其事必先利其器，只好先配置环境。

设备的CUDA是8.0版本的，最高只能支持TensorFlow 1.4的版本，但安装完后又发现这一版本不兼容设备默认装好的Python 3.6：
```
RuntimeWarning: compiletime version 3.5 of module 'tensorflow.python.framework.fasttensor_util' does not match runtime version 3.6
```

因为设备是公用的，不应该像个人电脑一样随意更改环境。在师兄的提示下，我意识到，解决Python环境配置难题的最好办法是使用Anaconda创建一个虚拟环境。

## 什么是Anaconda
简单地讲，Anaconda是一个开源包（packages）和虚拟环境（environment）的管理系统，它对虚拟环境的管理功能几乎就是为我遇到这种情况量身定做的。

## Anaconda使用示例
### 创建虚拟环境
第一步是根据自己的需求创建一个虚拟环境，基本命令是：
```
conda create -n env_name list_of_packages
```
其中，`env_name`指的是虚拟环境的名称，`list_of_packages`则是列出在新环境中需要安装的Python工具包。

例如，我打算创建一个Python 3.5的虚拟环境：
```
conda create -n py3.5 pip python=3.5
```
这里我将这个环境命名为`py3.5`，再指定Anaconda自动安装好pip和Python 3.5。

### 进入虚拟环境
创建好以后，运行下面的命令进入该环境：
```
source activate py3.5
```
然后看到shell的最前头出现了`(py3.5)`的标识符，表示我已经处在这个环境中：
![](https://controny.github.io/assets/images/posts/20180414142011.png)
同时查看Python的版本，确实是我想要的Python 3.5：
![](https://controny.github.io/assets/images/posts/20180414142132.png)

### 安装TensorFlow指定版本
接下来就只剩安装1.4版本的TensorFlow了：
```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple tensorflow-gpu==1.4
```
这里，我指定pip使用[清华大学的源](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)，同时指定安装TensorFlow GPU的1.4版。

### 退出虚拟环境
下面的命令用于退出当前环境：
```
source deactivate
```
可以发现，退出虚拟环境后，Python的版本变成了原来的3.6，说明上述环境是独立存在的。
![](https://controny.github.io/assets/images/posts/20180414143321.png)

## 参考
- <https://www.jianshu.com/p/169403f7e40c>
- <https://www.tensorflow.org/install/install_linux?hl=zh-cn#InstallingAnaconda>
