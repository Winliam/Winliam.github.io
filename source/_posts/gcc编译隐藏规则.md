---
title: gcc编译隐藏规则
categories:
  - - 工作技能
    - C/C++笔记
date: 2022-07-18 06:35:26
tags:
    - C++
    - 编译 
    - gcc
    - 链接
---
## 前言
[Linux下C++的编译、链接以及包管理](http://guohongming.cn/2022/07/17/Linux%E4%B8%8BC-%E7%9A%84%E7%BC%96%E8%AF%91%E3%80%81%E9%93%BE%E6%8E%A5%E4%BB%A5%E5%8F%8A%E5%8C%85%E7%AE%A1%E7%90%86/)一文中提到，Linux系统本身和GCC编译器在针对包的管理上存在一定的配合。

具体来说体现在，GCC在进行编译时会默认在特定的系统目录下去寻找头文件和库文件，而这些特定的系统目录恰恰是包安装的位置。

本文就展开说说这些默认规则。

## 基本规则
在介绍默认规则之前，先来回忆一下gcc命令行的基础规则有哪些：
```shell
gcc main.c function/head.c -I../header -L. -lvector -o main 
```
上面是一个非常典型的gcc使用场景，什么CMakelists.txt，makefile，catkin_make，bazel之类的编译管理工具无论搞得再花里胡哨，最后还是要落到这样的gcc调用上来。

先来翻译一下它的含义：以当前目录下的main.c和function目录下的head.c为源文件，将上一级目录下的header目录作为头文件搜索的**补充**路径，讲当前目录作为库搜索的**补充**路径，并寻找名为libvector.so或libvector.a的库文件参与编译，生成一个名为main的可执行文件。

要完成以上翻译，需要知道的规则有：
1. -I(大写i) flag用于添加头文件的**补充**搜索路径；
2. -L flag用于添加库文件的**补充**搜索路径；
3. -l(小写L) flag用于添加库文件的模糊搜索名称。之所以说是模糊搜索名称，是因为libvector.so，libvector.a都算匹配成功。如果刚好有重名的动静态库，那么优先选择动态库。
4. -o flag用于指定编译结果的名称；

这便是gcc最常用，最基础的规则。其中可以注意到，上面一直在强调**补充**二字，这是因为gcc有一些默认的头文件和库文件搜索路径，不管你有没有在调用gcc时显式地指定，它都是会按照一定的顺序去搜索这些默认路径。

## 隐藏规则
### 头文件搜索路径及顺序
1. 先在当前目录下找
2. 如果找不到，然后在-Iflag指定的目录找
3. 还是找不到，再到CPLUS_INCLUDE_PATH和C_INCLUDE_PATH两个环境变量指定的路径下去找。前者是g++用的，后者是gcc用的
4. 还是找不到，最后到/usr/include和/usr/local/include等系统目录中找，这里再找不到就编译失败了

### 库文件搜索路径及顺序——编译时
1. 先到-L指定的目录下找
2. 找不到去环境变量LD_LIBRARY_PATH指定的路径下去找
3. 还找不到就是去/lib和/usr/local/lib等系统目录中找，还找不到就链接失败了

### 动态库搜索路径及顺序——运行时
1. 先去环境变量LD_LIBRARY_PATH指定的路径下去找
2. 找不到去/etc/ld.so.cache文件中指定的路径下去找
3. 最后到/lib和/usr/local/lib等系统目录中找，还找不到就链接失败了

> ld.so.cache工作的机制是：在/etc/ld.so.conf.d/目录下有一些.conf文件（比如mylib.conf)，这些.conf文件的内容是该库的所在地址，比如mylib.conf的内容是/usr/local/me/mylib.so。运行`ldconfig`命令会读取所有这些.conf文件的内容，并将其写到ld.so.cache文件中。