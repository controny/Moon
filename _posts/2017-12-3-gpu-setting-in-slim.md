---
title: '在slim环境下设置gpu_option'
layout: post
tags:
- Tensorflow
- slim
- 深度学习
comments: true
---

## 设置`gpu_option`

使用Tensorflow的GPU版本进行深度学习训练时，Tensorflow默认占用对进程可见的所有GPU内存以提高训练效率。然而，在有些情况下，比如在多人共享的机器上训练时，系统管理员要求不能霸占所有GPU资源，否则其他人就无法使用了（博主就是这种情况），这时候我们就想要限制程序对GPU的使用，用英文讲，就是Allow GPU memory growth。根据[官网的说明](https://www.tensorflow.org/tutorials/using_gpu#allowing_gpu_memory_growth)，我们可以通过配置Session的`gpu_option`来达到上述目的。具体地，有两种参数可设置：
1. `allow_growth`
```
config = tf.ConfigProto()
config.gpu_options.allow_growth = True
session = tf.Session(config=config, ...)
```
上述代码的作用，简单地讲，就是按需分配：进程一开始运行的时候，仅为其分配少量的GPU内存，而随着进程的继续运行，对计算资源的需求加大时再分配更多的GPU内存。
2. `per_process_gpu_memory_fraction`
```
config = tf.ConfigProto()
config.gpu_options.per_process_gpu_memory_fraction = 0.4
session = tf.Session(config=config, ...)
```
顾名思义，这是在设置进程使用每个GPU的内存上限，例如，在上述代码里，该进程最多只会占用每个GPU内存的40%。

## 在slim环境下设置`gpu_option`

[slim](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/slim)是Tensorflow于2016推出的模块，是一种high-level库，讲许多深度学习算法封装起来，极大地简化了Tensorflow的代码。对于一般的神经网络训练，slim封装了[`slim.learning.train`](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/contrib/slim/python/slim/learning.py#L531)方法，程序员连Session都不需要自己创建了，然而，按照上述方法，`gpu_option`需要在创建Session时设置，这就产生了问题。所幸的是，`slim.learning.train`提供的参数已经考虑到了这个问题：

> session_config: An instance of `tf.ConfigProto` that will be used to
      configure the `Session`. If left as `None`, the default will be used.
     
 所以，我们的设置方法就是：
```
session_config = tf.ConfigProto()
session_config.gpu_options.allow_growth = True
slim.learning.train(..., session_config=session_config)
```
或者
```
session_config = tf.ConfigProto()
session_config.per_process_gpu_memory_fraction = 0.4
slim.learning.train(..., session_config=session_config)
```