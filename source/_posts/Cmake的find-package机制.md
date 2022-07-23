---
title: Cmake的find_package机制
categories:
  - - 工作技能
    - C/C++笔记
date: 2022-07-18 06:36:03
tags:
  - C++
  - Cmake
  - 编译
---
## 前言
本文是[Linux下C++的编译、链接以及包管理](http://guohongming.cn/2022/07/17/Linux%E4%B8%8BC-%E7%9A%84%E7%BC%96%E8%AF%91%E3%80%81%E9%93%BE%E6%8E%A5%E4%BB%A5%E5%8F%8A%E5%8C%85%E7%AE%A1%E7%90%86/)一文的支线，最好先去看下前文。

不看也没关系，Cmake的find_package机制也算是一个比较独立的话题。简单说就是，你从别的地方搞到自己机器上一个包之后，怎么通过Cmake来用它，或者说让它参与编译和链接？

先说结论，在CMakeLists.txt中find_package(XXX)被调用之后，后台程序去**一些路径**下查找名称中包含XXX，而且后缀为`.cmake`的文件。这个文件中定义了类似`${XXX_FOUND}`，`${XXX_INCLUDE_DIRS}`，`${XXX_LIB_DIRS}`这样的变量，用来指定XXX这个包是否被找到，包的头文件、库文件在本机上的绝对路径。这样，在CMakeLists.txt中又可以直接使用这些变量来进行头文件搜索和库的链接，elegant？！

大概的原理就是这样，不希望很深入的话了解到这里就可以了。想深入的话，就要聚焦于上面这段话中的两个模糊的地方：
1. **一些路径**具体指哪些？
2. 名称中包含XXX就可以？搜索时模糊匹配的规则是啥？

这两个问题其实就是，`.cmake`文件的搜索模式。先进行Module模式搜索，不成功再进行Config模式搜索。
## Module模式
该模式下搜索的目标是`FindXXX.cmake`文件，搜索的路径是：
1. 先到本机cmake的安装目录下的Modules文件夹下去找，比如我的是`/opt/cmake-3.23.2-linux-x86_64/share/cmake-3.23/Modules`，这下面是cmake预装的`.cmake`文件，包含了很多常用的库。
2. 再到CMakeLists.txt中显式指定的`${CMAKE_MODULE_PATH}`下去找，这意味着可以用户自定义搜索位置。

## Config模式
如果Module模式搜索失败，就开始进行Config模式搜索。该模式下搜索的目标是`XXXConfig.cmake`或者`xxx-config.cmake`(注意大小写)。搜索的路径一般是`usr/lib/cmake/xxx/`，`usr/local/lib/cmake/xxx`。

大多数库在通过`apt install`安装时，一般会在这两个搜索路径下自动生成这些`config.cmake`文件，比如eigen3。

