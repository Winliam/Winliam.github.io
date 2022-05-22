---
title: python列表的切片
categories:
  - - 工作技能
    - python笔记
date: 2021-03-29 19:08:41
tags:
  - python
  - 切片
  - 花式索引
---
普通索引`a[i]`返回序列对象的一个元素，切片`a[::1]`则返回一些元素。

## 基本索引
|a中元素 |2 |4 |6 |8 |
|:-:|:-:|:-:|:-:|:-:|
|非负索引 | 0|1 |2 |3 |
|负数索引 |-4 |-3 |-2 |-1 |
> 非负索引从0开始，负数索引从-1开始
> 基本索引不可越界

## 简单切片
`a[start:stop]`返回基本索引范围在[start,stop)内的元素
  1. start/stop越界：此时有多少算多少，不报错
  2. start > stop：返回空即可
  3. start/stop缺省：start默认无穷小，stop默认无穷大

## 扩展切片

#### step为正数，从前往后选取
`a[start:stop:step]`在[start,stop)区间内，依次选取索引为start, start + step, start + 2step.....直到索引值不在[start, stop)区间中
  1. 越界：同样有多少算多少，不报错
  2. start > stop：返回空
  3. 缺省：同上

#### step为负数，从后向前选取
此时，start是stop，stop是start，而且还是左开右闭，比如
```python
a[2:0:-1] -> [6, 4]

a[:1:-2] -> [8]
```

## 多维数组切片
numpy和tensor这样的数据类型相比于原生的list提供了多维数组切片方法，示例如下：
```python
pad_img[: img.shape[0], : img.shape[1], : img.shape[2]].copy_(img)
```
两个逗号隔开三个索引表达式，分别对应这个图像/tensor/numpy数组的3个维度，各个索引表达式在各自维度上的切片规则同上，得到各个维度上的索引列表后对3个列表的元素进行全排列，即可得到最终的三维索引(x, y, z)。这个例子的含义是把img中的像素值复制到pad_img中，左上角对齐。

研究SSD的源码时又发现一个tensor的切片方式：
```python
bboxes = [[0, 1, 2], [3, 4, 5], [6, 7, 8]]

a = bboxes[[True, False, True], 0]
b = bboxes[[1, 0, 1], 0]

>>> a = [0, 6]
>>> b = [3, 0, 3]
```
基本的规则还是跟第一个例子一样，先获取各个维度的索引列表，然后进行全排列，得到最终的二维索引(x, y)。但是第一个维度上的索引列表是一个bool类型元素的列表，当取Fasle时，不参与索引。
