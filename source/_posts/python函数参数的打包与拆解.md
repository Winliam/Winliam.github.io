---
title: python函数参数的打包与拆解
categories:
  - - 工作技能
    - python笔记
date: 2019-07-29 18:14:46
tags:
  - python
---
**标志**：单/双星号出现在入参或形参之前
**原则**：
1. 入参前加星号代表拆解，形参前加星号代表打包
2. list/tuple只有一种拆解方式，dictionary有两种

## 打包情形1：单星号出现在形参前
```
def pack(a, *b):
        print type(a), a
        print type(b), b
	
pack(1, 2, 3, 4, 5)

>> <type 'int'> 1
>> <type 'tuple'> (2, 3, 4, 5)
```
单星号打包，按照匹配顺序认领多个非关键字入参，打包称一个tuple。

## 打包情形2：双星号出现在形参前
```
def pack(a, **b):
        print type(a), a
        print type(b), b
 
pack(1, a1=2, a2=3)

>> <type 'int'> 1
>> <type 'dict'> {'a1': 2, 'a2': 3}
```
双星号打包，按照匹配顺序认领多个关键字入参，打包成一个dictionary

## 拆解情形1：单星号出现在入参前
```
def pack(a, b, *c):
        print type(a), a
        print type(b), b
        print type(c), c
 
score = [1.0, 2.0, 3.0, 4.0]
pack(*score)

>> <type 'float'> 1.0
>> <type 'float'> 2.0
>> <type 'tuple'> (3.0, 4.0)
```
单星号拆解，将list/tuple/dictionary的内容当做独立非关键字的入参列表，其中dictionary只保留key

## 拆解情形2：双星号出现在入参前
```
def pack(*a, **b):
        print type(a), a
        print type(b), b
 
age = [1, 2, 3]
student = {'score' : 1.0, 'id' : 2, 'name' : 'xiaoxiao'}
pack(*age, **student)

>> <type 'tuple'> (1, 2, 3)
>> <type 'dict'> {'score': 1.0, 'id': 2, 'name': 'xiaoxiao'}
```
双星号拆解，只针对dictionary，将其内容拆解为关键字入参列表。
