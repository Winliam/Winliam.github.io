---
title: pytorch基础用法-预测
categories:
  - - 工作技能
    - 神经网络
    - pytorch使用
date: 2020-05-11 17:52:15
tags:
  - pytorch
---
## 数据准备
```
transform = transforms.Compose(
        [transforms.Resize((32, 32)),
         transforms.ToTensor(),
         transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5))])

im = Image.open('1.jpg')
im = transform(im)  # [C, H, W]
im = torch.unsqueeze(im, dim=0)  # [N, C, H, W]
```

## 模型定义
```
# 不需要to(device)吗？mark
net = LeNet()
```

## 参数加载
```
# 优化器也有同样的操作，可用于恢复训练
net.load_state_dict(torch.load('Lenet.pth'))
```

## 前向计算
```
with torch.no_grad():
    outputs = net(im)
    predict = torch.max(outputs, dim=1)[1].data.numpy()
```
