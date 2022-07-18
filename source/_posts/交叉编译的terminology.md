---
title: 交叉编译的terminology
categories:
  - - 工作技能
    - C/C++笔记
date: 2022-04-19 14:35:50
tags:
  - 交叉编译
  - cross building
---
## 前言
交叉编译的概念不难理解，就是在一台A操作系统的环境下，编译出用于在B操作系统下运行的二进制文件。

但是今天看[conan的文档](https://docs.conan.io/en/1.34/systems_cross_building/cross_building.html#cross-building)时，发现GNU对于交叉编译的术语有一些约定，而且还挺绕，所以整理记录一下。

## GNU convention
### Different Platforms
- **Build Platform**，编译这个过程发生在哪个操作系统上，哪个操作系统就是**Build Platform**。
- **Host Platform**，编译的产物运行于哪个操作系统上，哪个操作系统就是**Host Platform**。
- **Target Platform**，由于编译器本身也是一个程序，所以在编译“编译器”，尤其是在编译“交叉编译器”时，才会用到**Target Platform**的概念，后面结合例子细讲。
### Different Building Processes
- **Native Building**，`build platform`和`host platform`相同的编译过程，就叫做**Native Buildig**。
- **Cross Building**，`build platform`和`host platform`不同的编译过程，就叫做**Native Buildig**。

## Examples
举一个最复杂的例子，来帮助理解。现在有一个编译任务是：

**AndroidNDK是一个运行在Linux上的编译器，用来产出运行于Android系统的可执行文件。现在我们想在Windows系统下将AndroidNDK的源码编译成可执行文件。**

在从AndroidNDK源码到可执行文件的编译任务中，
- Build Platform = Windows，因为编译过程发生在Windows上
- Host Platform = Linux，因为编译产出物（AndroidNDK可执行文件）跑在Linux上
- Target Platform = Android，因为编译产出物的编译产出物跑在Android上
