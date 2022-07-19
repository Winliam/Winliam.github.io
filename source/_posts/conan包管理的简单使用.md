---
title: conan包管理的简单使用
categories:
  - - 工作技能
    - C/C++笔记
date: 2022-07-18 06:36:25
tags:
  - conan
  - C++包管理
  - Cmake
  - 编译
---
## 前言
本文是[Linux下C++的编译、链接以及包管理](mark)一文的支线，最好先去看下前文。

简单说conan就是一个C++的包管理工具，包含包的封装、存储、发布、使用等一整套体系。

其实每一种编程语言都有自己包管理工具，比如python的pip，javascript的npm，java的maven等等。但是C++[很特殊](https://www.zhihu.com/question/26117075/answer/86056267)，它的包管理工作比较难做，基本没有一统江湖型的解决方案。

conan相对而言是一个比较成功的尝试，现公司也在主推这个，所以就简单介绍下conan的基础功能。（主要是复杂的功能还没机会参透，后面慢慢更新吧）

其核心的思路是:
1. 一个包的作者在完成代码的编写和编译之后，将编译时的一些环境信息（比如arm架构下基于gcc8编译的）记录下来连同代码一起进行封装，然后放到服务器上供别人下载。
2. 别人在下载这个包时可以向服务器发出包含特定的配置信息的请求来下载包（比如要基于arm架构，用gcc8编译的3.4.0版本的opencv）
3. 包被下载到本地后（默认时存放在用户目录下的`.conan`隐藏目录中），包的使用者可以用conan提供的配套工具来生成`.cmake`这样的编译辅助文件，然后结合Cmake的find_package机制便可以正常使用了。

> conan是一个由python实现的包，所以安装时使用pip，各种配套命令行的后端也都是python在做解析。

## Terminology
名不正则言不顺，首先还是来整理一下conan关键的概念：
- **package**：即[之前](mark)说过的一个包，主要由库和同文件组成。
- **recipe**：实体是一个conanfile.py，每个包一个，在里面实现了一个辅助包管理的类；这个类的成员用来记录这个包的属性信息（settings和options），类的方法则对应编译、打包、上传等包管理操作。在终端中使用conan系列命令时，其实就是在调用这个方法。
- **package id**：在进行打包操作时，conan会检查包当前recipe中的settings和options属性，将它们hash成一个字符串，作为这个包针对这张recipe的唯一标识；
- **paackge reference**: 在conan的语境中，如何引用一个包（比如在为一个包指定其依赖时），格式是`package_name/version@user/channel`，例如`w3_opencv/4.2.0@pjw3/dev`，其中user和channel就是单纯的标识符，没太多含义。
- **profile**：将自己当前开发所需的库的配置记录下来，当向服务器请求包时可以直接将profile发过去，告诉服务器我要什么配置的包。
- **registry**：存放包的remote服务器，一般是基于artifactory搭建。

> recipe的实体也可以是conanfile.txt，但用的比较少，不做介绍了

## conanfile.py中辅助类的常用[属性](https://docs.conan.io/en/latest/reference/conanfile/attributes.html)
- **name**: 包的名称（即package _name)
- **version**: 包的版本
- **settings**：记录诸如这个包中的库是在什么操作系统上编译的，基于哪一版c++标准，用的什么编译器这种信息；总的来说是project-wide的属性
- **options**: 记录一些编译选项，例如debug编译还是release编译，变成动态库还是静态库这种；总的来说是package-specific的属性
- **generator**：conan不只支持Cmake的find_package机制进行包的使用，也支持bazel等其他机制，这个属性就是用来告诉conan你打算咋用这个包，它好给你生成对应的编译辅助文件。我目前只用到了`cmake_find_package`这一种。

## conanfile.py中辅助类的常用方法
(mark)

## 常用命令
(mark)