---
title: ResNet
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
  - ResNet
  - Residual结构
date: 2020-10-08 15:30:36
---

## 背景简介
来自2015年的微软实验室，He Kaiming/Ren Shaoqing/Zhang Xiangyu等，啥啥啥都第一名...不用dropout了，引入BN，提出residual以应对退化问题，支持上千层的网络结构。

## 网络结构
### 整体结构
网络由一种叫做residual的building block串行组建而成，整体结构见下表：
<div align=center><img title="" src="/img/net/resnet.png" width="100%" height="100%" align=center></div>

### 常规的Residual单元
如下图所示，常规的Residual有两种规格，前者用于层数较少的网络，后者用于层数较多的网络。

<div align=center><img title="" src="/img/net/residual1.png" width="80%" height="80%" align=center></div>

Residual特点如下：
1. 为了实现shortcut结构，必须保证单元的输入与输出尺寸完全一致，包括维度。这样才能做tensor的逐元素相加。
2. 最后一个relu是在逐元素相加之后进行的。
3. 和Inception单元一样，每个卷积层的FSP都是确定的，其中F看位置取1或3，S=1，P=SAME PADDING。因此一个Residual单元的配置参数只有每个卷积层的#。

### 特殊的Residual单元
在分界线处的Residual单元还要额外承担tensor尺寸调整的任务，因此其结构相对于常规结构要增加一些变化。
<div align=center><img title="" src="/img/net/residual2.png" width="60%" height="60%" align=center></div>
<div align=center><img title="" src="/img/net/residual3.png" width="60%" height="60%" align=center></div>

首先是在shortcut上增加一个1F2S0P?#的卷积层对单元输入进行尺寸调整。另外是主线上的第一个卷积层的S变为2，进行尺寸调整。
> 上图是按照pytorch中的实现，是在第2个卷积层做的下采样，实测效果更好，所以这么用了。

## 其他内容
1. **Residual结构为什么有效？**
  xxxx
