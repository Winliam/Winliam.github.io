---
title: MobileNet v2
math: true
categories:
  - - 工作技能
    - 神经网络
    - 经典网络
tags:
  - 深度学习
  - 神经网络
  - 卷积神经网络
  - CNN
  - 经典网络
  - MobileNet
  - Residual结构
  - Inverted Residual结构
date: 2020-09-20 22:22:19
---
## 背景简介
2018年Google团队在v1的基础上提出的改进版。核心思想仍然是DW卷积，但是这次借鉴了ResNet的残差结构，提高了性能。

## 前置知识
### Inverted Residual
所谓倒残差结构，其实就是在ResNet残差结构的基础上做了一些修改。首先看以下简图：
<div align=center><img title="" src="/img/net/ir.png" width="70%" height="70%" align=center></div>

<br>对于残差结构而言，三个卷积层每一层卷积核的个数一般依次为[64, 64, 256]这样，先对具有256个通道的输入进行降维，然后3x3卷积 with SAME PADDING, 尺寸和通道都不变，最后再用1x1卷积进行升维。就三个卷积层的4个计算对象而言，通道数依次为[256, 64, 64, 256]，两头大中间小。

对于倒残差结构则刚好相反，它先升维，再用DW卷积保持深度，然后再降维输出，呈现两头小中间大的形状，因此称之为倒残差。

具体来看倒残差结构的结构和参数，如下表所示：
<br><div align=center><img title="" src="/img/net/ir2.png" width="60%" height="60%" align=center></div><br>

1. 第一层为PW卷积+BN+RELU6，其中卷积层参数为1F-1S-SP-tk#，k为单元输入的通道数；
2. 第二层为DW卷积+BN+RELU6，参数为3F-sS-SP, 当s=1或2；
3. 第三层为PW卷积+BN+Linear，卷积层参数为1F-1S-SP-k'#，k'为单元输出的通道数；

综上，一个倒残差单元的参数有k，t，s，k'四个。4个计算对象的深度分别为[k, tk, tk, k']。

### Relu6激活函数
倒残差结构相对于残差结构的另外一个不同点，其实就是在x=6处饱和的Relu函数。

### Linear激活函数
倒残差单元输出之前进行的不是Relu激活，而是线性激活。因为，据称，在对通道数较少的tensor进行激活时，Relu会丢失更多信息。而每个倒残差单元又是两头小，因此最后的激活函数使用了简单的线性函数。另外，由于线性激活函数的表达式就是$y = x$，所以线性激活=不做激活。

## 网络结构
整个网络来看，先用倒残差单元串联组成单元层，单元层和普通卷积层、平均池化层、FC层再串联组成网络。

<div align=center><img title="" src="/img/net/mbv2.png" width="50%" height="50%" align=center></div>

看懂上表首先要明确一个单元层的4个参数t,c,n,s的含义：
- t, 对应倒残差单元的参数t
- c, 对应本单元层中的第一个倒残差单元的k'参数，以及后续的倒残差单元的k和k'参数。就是说对于一个单元层，计算对象的深度依次为[input, input x t, input x t, c]、[c, c x t, c x t, c]、[c, c x t, c x t, c]......
- n, 将几个倒残差单元串联起来形成本单元层
- s，对应本单元层中第一个倒残差单元的s参数，其余单元的s参数都为1

另外，还要注意以下3个细节：
1. 每个单元层的第一个倒残差单元与ResNet中的处理方式不一样，这里是放弃shortcut而不是在shortcut上插入一个1x1卷积来进行深度的一致化。
2. 当每个单元层的中间单元，不满足输入输出的tensor形状、深度一致的条件时，也不进行shortcut。
3. 对于t=1的单元层，取消其倒残差单元的第一个PW卷积，因为通道数量并不需要变化。


## 其他内容
1. 倒残差结构的三个特点为什么work？mark
1. DW卷积在pytorch中的实现是通过控制`nn.conv2D()`构造函数中的`groups`参数来实现。`groups = 1`时就普通的卷积，`groups = input_chs`时就是DW卷积。
