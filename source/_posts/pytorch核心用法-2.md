---
title: pytorch基础用法-训练
categories:
  - - 工作技能
    - 神经网络
    - pytorch使用
date: 2020-05-11 17:52:12
tags:
  - pytorch
---
## 数据准备
### 引入数据集
```
# 图片在送入网络之前要进行一定的变化才能适应网络入口的要求，比如尺寸一致化，张量化，归一化等
data_transform = {
        "train": transforms.Compose([transforms.RandomResizedCrop(224),
                                     transforms.RandomHorizontalFlip(),
                                     transforms.ToTensor(),
                                     transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5))]),
        "val": transforms.Compose([transforms.Resize((224, 224)), 
                                   transforms.ToTensor(),
                                   transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5))])}

# 用pytorch官方的数据集，本地没有就去下载，需要指明是用数据集中的train部分，还是val部分
train_set = torchvision.datasets.CIFAR10(root='./data', train=True, download=True, transform=transform)
val_set = torchvision.datasets.CIFAR10(root='./data', train=False, download=False, transform=transform)

# 用自定义的数据集，前提是本地先下载好，并且建立好目录结构:
# image_path
# 	- train
# 		- class1
#			- 1.png
#			- 2.png
# 		- class2
#			- 1.png
#			- 2.png
# 	- val
# 		- class1
#			- 1.png
#			- 2.png
# 		- class2
#			- 1.png
#			- 2.png
train_dataset = datasets.ImageFolder(root=os.path.join(image_path, "train"), transform=data_transform["train"])
validate_dataset = datasets.ImageFolder(root=os.path.join(image_path, "val"), transform=data_transform["val"])

# 最后得到的是一个torch定义的class，按[]索引得到的是一个tuple，tuple元素是(tensor,id)，每个tensor对应一个张量化后的图片。
```
### 加载数据
```
# 用num_workers个线程，将train_set打乱顺序，分成36个一份的batch
train_loader = torch.utils.data.DataLoader(train_set, batch_size=36, shuffle=True, num_workers=0)

# 最后得到也是一个torch定义的class，被enumerate返回一个[tensor,tensor]
# 第一个tensor代表inputs，维度为[batch_size, c, h, w]
# 第二个为labels，维度为[batch_size]
# 也可以用iter将其加工成一个可迭代对象来访问其内容
train_data_iter = iter(train_loader)
img, label = train_data_iter.next()
```

## 工具定义
```
from model import LeNet
import torch.optim as optim

# 模型定义
net = LeNet()
net.to(device)

# 损失函数定义
loss_function = nn.CrossEntropyLoss()

# 优化器定义
# 注意这里的优化器不是按照SGD/BGD/MBGD这样实现的，因为这种分类方式依赖的分类标准是batch_size的值。也就是说这里的SGD其实是基本的GD，然后如果实际的batch_size是1，那么才是真正的SGD。而且如果加上momentum参数，那么就是GD with Momentum。在定义优化器是需要指定的其实是参数更新的方式，而不是每次送入几个样本。
params = [p for p in model.parameters() if p.requires_grad] #可以在这里之前将某些层的参数锁定，从而实现冻结某些层不训练的效果
optimizer = optim.Adam(params, lr=0.0002)
optimizer = optim.SGD(params, lr=0.0002, momentum=0.1)


# 学习率调整器定义
# 在整个训练过程中，学习率保持一个定值效果往往不太好。因此一般用这个调整器来规划学习率随着epoch数量增加的变化策略。一种流行的做法是三角学习率，即学习率先增后减。开始阶段过大容易直接跑飞，结束阶段过大很难收敛，中间过程太小又会导致损失下降太慢，因此先增后减。

def f(x):
     if x >= warmup_iters:  # 当迭代数大于给定的warmup_iters时，倍率因子为1
         return 1
     alpha = float(x) / warmup_iters
     # 迭代过程中倍率因子从warmup_factor -> 1
     fx = warmup_factor * (1 - alpha) + alpha
     return fx
lr_warmpuper = torch.optim.lr_scheduler.LambdaLR(optimizer, lr_lambda=f) #第i次调用step方法时会将optimizer的学习率变为initial_alpha*f(i)
lr_scheduler = torch.optim.lr_scheduler.StepLR(optimizer, step_size=3, gamma=0.33) #每调用3次step方法，optimizer的学习率乘以一次0.33
```

## 核心循环
```
for epoch in range(epochs):
# 训练循环
        # 设置train模式，以打开BN，Dropout等训练专用的运算
        net.train()
        for data in train_loader:
            images, labels = data
            # 重置梯度变换量，防止过大？mark
            optimizer.zero_grad()
            # 正向计算
            outputs = net(images.to(device))
            # 计算损失
            loss = loss_function(outputs, labels.to(device))
            # 反向传播
            loss.backward()
            # 梯度更新 
	    optimizer.step()
	    if epoch==0:  # 第一轮使用warmup训练方式，每一个iteration调整一次学习率
               lr_warmuper.step()         
	lr_scheduler.step()
# 验证循环
        # 设置eval模式，以关闭BN，Dropout等训练专用的运算
        net.eval()
        # 验证过程不需要自动存储梯度？mark
        with torch.no_grad():
            for val_data in val_loader:
                val_images, val_labels = val_data
                outputs = net(val_images.to(device))
                # 求softmax向量的最大值的索引作为预测结果。
                predict_y = torch.max(outputs, dim=1)[1]
# 模型保存
        if val_accurate > best_acc:
            best_acc = val_accurate
            # 只保存权重，官方推荐这一种
            torch.save(net.state_dict(), save_weight_path)
            # 同时保存和网络
            torch.save(net, save_model_path)
```
