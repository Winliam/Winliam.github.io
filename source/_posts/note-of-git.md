---
title: note_of_git
categories:
  - - 工作技能
    - git笔记
date: 2021-05-22 18:38:33
tags:
  - git
  - .gitignore
---
# 前言
天天都要跟git打交道，但是这玩意说实话有些复杂。以具体场景为记录方式，整理下git的一些经典使用方法。

## 在`.gitignore`中排除忽略
忽略一个文件或者目录很容易，加到`.gitignore`文件中即可，匹配模式啥的也不难，百度一下就能上手。但是如何做到在忽略某个目录的同时，又不忽略该目录下的某个子目录呢？
```
# 先忽略node_modules目录下的所有内容，但是不忽略node_modules目录本身，否则下面咋搞都不行了
node_modules/*
# 取消对node_modules目录下hexo-theme-fluid的忽略
!node_modules/hexo-theme-fluid/
node_modules/hexo-theme-fluid/*
!node_modules/hexo-theme-fluid/languages/
```
总结下来就是，最后的*很关键，如果不加它，git会直接把目录本身也忽略，后面即使再取消忽略子目录也不行了，因为git在解析node_modules的目录结构之前就把它忽略了，根本不知道它还有hexo-theme-fluid这么个子目录。