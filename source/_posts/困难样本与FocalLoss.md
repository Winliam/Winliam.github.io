---
title: 困难样本与FocalLoss
categories:
  - - 工作技能
    - 神经网络
date: 2020-12-13 09:25:31
math: true
tags:
  - 损失函数
  - Focal Loss
  - 困难样本
---
## 前言
one-stage的目标检测算法，一般都存在正负样本数量不均衡的问题。

回忆一下，正样本指的是，对于一个anchor(记作A)，为其寻找最佳GTbox，也就是与其IOU最大的GTbox(记作B)。然后，计算A和B的IOU，如果超过某个阈值，则认为A和B之间的匹配关系为一个正样本(也就是说正负描述的都是一个match)，并将B的label赋予A。否则，视作负样本，A的label为背景。而且由于负样本没有对应的GTbox，所以一般不参与定位损失的计算，是否参与分类损失则没有定论。

## 困难样本

之前提到的SSD缓解正负样本不平衡所使用的方法是，按照造成的损失大小对负样本排降序，然后取topK计入最终的损失。其中，K是正样本数量的3倍。

这是一种Hard Negative挖掘的方法。那么这里Hard的含义是什么呢？直观描述就是在分类损失的计算中，那些预测分数很高的logit，竟然不是GT one-hot向量hot的位置；或者那些预测很低的logit，竟然不是GT one-hot向量中不hot的位置。

<div align=center><img title="" src="/img/article/hard_match.png" width="80%" height="80%" align=center></div>

<br>显然，困难样本无论是用交叉熵还是二值交叉熵计算时，都会产生更大损失，应该更充分地被纳入损失计算，这也是SSD上述做法的内在逻辑。

## Focal Loss
Focal Loss则提出了一种更加系统化的解决方案，先看普通的二值交叉熵损失计算公式：
$$
L = -{\rm ln}\hat{p} \;\;\;\;\;\; {\rm if \;}p = 1
$$
$$
L = -{\rm ln}(1-\hat{p}) \;\;\;\;\;\; {\rm if \;}p = 0
$$
Focal Loss对其做了这样一种加工，变为
$$
L = \alpha(1-\hat{p})^\gamma \cdot -{\rm ln}\hat{p} \;\;\;\;\;\; {\rm if \;}p = 1
$$
$$
L = (1-\alpha)\hat{p}^\gamma \cdot -{\rm ln}(1-\hat{p}) \;\;\;\;\;\; {\rm if \;}p = 0
$$
也就是在每一项上乘以一个增益，进一步放大困难样本本来就比较大的损失，从而加速模型的收敛。

例如当GT值$p=1$时，显然预测值$\hat{p}$越小，这个match越困难。此时相应地，$(1-\hat{p})^\gamma$也越大，L被放大。当GT值$p=0$时则相反，预测值$\hat{p}$越大越困难，此时$\hat{p}^\gamma$也相应地越大，L被放大。

而$\alpha$则用于平衡这两种情况对损失贡献的权重。

## 注意事项
使用Focal Loss机制时要小心错误标注样本，因为这些错误标注的GT值会被Focal Loss机制当做困难样本，进一步放大其影响。
