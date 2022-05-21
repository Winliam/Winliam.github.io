---
title: CMake笔记
categories:
  - - 工作技能
    - C/C++笔记
date: 2022-04-05 13:06:45
tags:
  - CMake
  - INTERFACE
---
## CMake三大原则
- **Decalare a target**;
- **Decalare traits of the target**;
- **It is all about target**.

## PUBLIC PRIVATE INTERFACE关键字
### 前言
这三个关键字出现在`target_include_directories`和`target_link_libraries`两个命令中，如
```
add_library(fruit STATIC fruit.cpp)
target_include_directories(fruit PUBLIC include/fruit_h/)

add_library(apple STATIC apple.cpp)
target_include_directories(apple PUBLIC include/apple_h/)
target_link_libraries(apple INTERFACE fruit)


add_library(eat_apple STATIC apple.cpp)
target_include_directories(eat_apple PUBLIC include/eat_apple_h/)
target_link_libraries(eat_apple PRIVATE apple)
```
### 在头文件命令中的作用
首先，借助两个预定义变量，或者说target默认拥有的两个traits`INCLUDE_DIRECTORIES`和`INTERFACE_INCLUDE_DIRECTORIES`来说明PRIVATE等关键字在`target_include_directories`命令中的作用。
- `INCLUDE_DIRECTORIES`中存放的是当前target的头文件搜索路径
- `INTERFACE_INCLUDE_DIRECTORIES`用于存放的是另外一组头文件搜索路径，在编译本target（记作A）时不生效。当`target_link_libraries(B PUBLIC A)`命令执行时，会将A的`INTERFACE_INCLUDE_DIRECTORIES`中的内容append到B的`INCLUDE_DIRECTORIES`中。

然后，在`target_include_directories`命令中的PRIVATE等关键字通过影响这两个traits来产生效果：
- PRIVATE会将其后的路径添加到当前target的`INCLUDE_DIRECTORIES`中；
- INTERFACE会将其后的路径添加到当前target的`INTERFACE_INCLUDE_DIRECTORIES`中；
- PUBLIC则同时添加；

### 在链接命令中的作用
链接命令中虽然没有（尚待确认）类似`INCLUDE_DIRECTORIES`和`INTERFACE_INCLUDE_DIRECTORIES`这两个显式的traits，但是最终效果是一样的，可以按照存在`LINKED_LIBRARIES`和`INTERFACE_LINKED_LIBRARIES`这么两个traits，然后结合上述机制来理解。

然后，分析`target_link_libraries(apple INTERFACE fruit)`的情况。此时apple并未真正的链接fruit库，也就是说如果apple中调用了一个实现在fruit中的函数size()，那么编译是会报错的，但是eat_apple中size()却是可以的。

最后分析`target_link_libraries(apple PRIVATE fruit)`的情况。此时apple中可以调用size()，但是eat_apple中却不行了，因为此时fruit被添加到了apple的`LINKED_LIBRARIES`中，但是没有被添加到apple的`INTERFACE_LINKED_LIBRARIES`中，自然也就不会被append到eat_apple的`LINKED_LIBRARIES`中。

### 结论
- **PRIVATE只关心本target**
- **INTERFACE只关心依赖本target的target**
- **PUBLIC同时关心**

### 参考链接
1. https://leimao.github.io/blog/CMake-Public-Private-Interface/#:~:text=PRIVATE%20only%20cares%20about%20himself,about%20everyone%20and%20allows%20inheritance.
2. https://kubasejdak.com/modern-cmake-is-like-inheritance
