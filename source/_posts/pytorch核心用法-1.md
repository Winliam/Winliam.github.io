---
title: pytorch基础用法-模型
categories:
  - - 工作技能
    - 神经网络
    - pytorch使用
date: 2020-05-11 16:52:15
tags:
  - pytorch
  - python
hide: false 
---

搭建一个CNN模型用作图像分类的过程，整体而言还是比较清晰的，主要分为模型、训练、预测三项工作。

第一项工作，模型搭建，套路也很清晰。新建一个继承于`nn.Module`的类，然后
1. 将网络层一一作为类的成员添加进来。这些网络层可以是`Conv2d`, `MaxPool2d`, `Linear`等`torch.nn`提供的基本模块，也可以是由这些基本模块组合成的自定义building blocks，如ResNet中的Residual单元，GoogLeNet中的Inception单元等。
2. 定义网络层之间的连接关系，也就是`forward`函数的行为。

另外，一种常见的做法是在定义网络层成员的时候，直接使用`nn.sequential`函数将多个网络层连接模块（比如一个classifier和一个features），然后在`forward`中连接这些模块即可。<br>

BN, Dropout, Relu都可以作为一个网络层存在和连接。

```python
import torch.nn as nn
import torch.nn.functional as F

class LeNet(nn.Module):
    # 设定有哪些网络层，每层的尺寸
    def __init__(self):
        super(LeNet, self).__init__()
        self.conv1 = nn.Conv2d(3, 16, 5)
        self.pool1 = nn.MaxPool2d(2, 2)
        self.conv2 = nn.Conv2d(16, 32, 5)
        self.pool2 = nn.MaxPool2d(2, 2)
        self.fc1 = nn.Linear(32*5*5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    # 设定层与层之间的怎样连接
    def forward(self, x):
        x = F.relu(self.conv1(x))    # input(3, 32, 32) output(16, 28, 28)
        x = self.pool1(x)            # output(16, 14, 14)
        x = F.relu(self.conv2(x))    # output(32, 10, 10)
        x = self.pool2(x)            # output(32, 5, 5)
        x = x.view(-1, 32*5*5)       # output(32*5*5)
        x = F.relu(self.fc1(x))      # output(120)
        x = F.relu(self.fc2(x))      # output(84)
        x = self.fc3(x)              # output(10)
        return x
```

