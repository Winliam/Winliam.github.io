---
title: Shell笔记
categories:
  - - 工作技能
    - shell笔记
date: 2019-12-25 17:13:22
tags:
  - shell
  - bash
  - 重定向
  - nohup
  - 后台
math: true
---
### shell解释器的种类
1. 一个shell脚本可以被不同的解释器解释执行，常见的解释器有sh、bash、zsh、dash。可以使用以下命令查看本机的shell解释器配置：
  ```shell
  #本机可用的shell解释器
  cat /etc/shells
  
  #当前默认设置，一般在第一行
  cat /etc/passwd

  #当前设置
  echo $SHELL
  ```
2. 一个shell脚本具体被哪一个解释器解释执行，依次看：
  - cmd命令指定的是哪个
  - shebang指定的是哪个
  - [都不指定的话不清楚用哪个](https://stackoverflow.com/questions/69643212/how-is-it-determined-which-shell-runs-a-script?noredirect=1#comment123105552_69643212)
  ```shell
  #cmd命令指定
  bash ./test.sh
  zsh ./test.sh

  #Shebang指定
  #!/usr/bin/env bash		到PATH路径下找到bash以执行脚本
  #!/bin/bash			用/bin目录下的bash程序执行脚本

  #暂时不清楚用哪个shell来执行
  ./test.sh			
  ```

### shell脚本的执行方式
shell脚本的执行方式有source和bash两类，前者的一般用途是实现python的import机制，即在另一个shell脚本中实现一堆公用的变量和函数，然后加载到当前脚本中。而后者则是真实的执行。

When you execute the script you are opening a new shell, type the commands in the new shell, copy the output back to your current shell, then close the new shell. Any changes to environment will take effect only in the new shell and will be lost once the new shell is closed.
 
When you source the script you are typing the commands in your current shell. Any changes to the environment will take effect and stay in your current shell.

Sourcing a script will run the commands in the current shell process. Changes to the environment take effect in the current shell.

Executing a script will run the commands in a new shell process. Changes to the environment take effect in the new shell and is lost when the script is done and the new shell is terminated.

Use source if you want the script to change the environment in your currently running shell. use execute otherwise.

### 重定向
```shell
cmd > file 2>&1
# 首先将cmd的输出重定向到file中，然后将标准错误输出绑定到标准输出上。合起来的意思是将cmd产生的，本来应该分别输出到标准输出和标准错误输出的打印信息，全部输出到file文件中

```
```shell
cmd &>/dev/null
cmd >/dev/null 2>&1
# 这两句话等价，那么从第二句看，并且结合前面的内容可以知道，这里的意思是将cmd产生的stdout和stderr都重定向到/dev/null中。这个/dev/null可以看作一个blackhole，会把所有输进去的内容吃掉。因此，这句话一般用作忽略cmd的任何输出信息。
```
### 静默执行
涉及到的概念有：**前/后台程序**，**nohup前缀**，**&后缀**，**Ctrl-Z**，**bg/fg命令**，**HUP信号**，**TSTP信号**。
#### 前/后台程序
前/后台程序的区别在于，启动该程序的shell是否被该程序阻塞。阻塞，意味着shell暂时不能跟用户交互了，只能等待前台程序执行结束，后台则无此影响。

在原本的启动命令后加上 & 后缀，即可将该程序放到后台执行。并且，若此时未经过重定向，还是会将打印信息输出到终端，所以要想完全放到后台还需要将输出重定向。
  ```shell
  yes love &>/dev/null &
  ```

当一个程序已经在前台阻塞式执行时，使用Ctrl-Z命令可以向程序发送TSTP信号，将程序暂停。然后通过bg命令可以将该程序恢复到后台执行，fg则恢复到前台。
#### 防挂起
nohup前缀产生的相关效果是独立于前/后台程序的另一个话题，其直接效果是使得程序免疫HUP信号。HUP信号一般会在关闭终端或用户注销时发出，接收到HUP信号的程序会终止执行。间接效果是将程序的stdout和stderr信息重定向到一个nohup.out文件。

综上，用下面这个形式的命令可以实现一个程序的**安全(终端关掉也不会停)**、**静默(启动了就跟没启动一样)**执行。
```shell
nohup cmd &>/dev/null &
```

[参考链接](https://linuxhint.com/how_to_use_nohup_linux/)

### 命令选项结束符`--`
思考一个问题，如果想在readme.txt中查找字符串"-v"，应该怎么做？

直接`grep -v readme.txt`是不对的，因为`-v`是`grep`命令用于反选的选项。因此，要用到`--`符号，表示command options结束，positional argument开始。

即，`grep -- -v readme.txt`。

### 自动补全
想必每个linux user对于TAB键有着特殊的感情，那么这种TAB自动补全是怎么实现的呢？以及对于自己写的一些可执行文件，又怎么给它安排上自动补全的功能呢？

首先介绍以下关键字：
- `complete`
  shell内置的一个命令，通过给入不同的flag，来给指定的可执行程序加上不同的自动补全内容。常用的有：
  ```shell
  # dothis <Tab><Tab>后，从Wordlist中选取合适的补全内容
  complete -W "now tomorrow never" dothis

  # dothis <Tab><Tab>后，执行_dothis_completions函数，按照该函数的行为进行补全
  complete -F _dothis_completions dothis
  ```
- `COMPREPLY`
  shell内置的一个变量，用来保存`<TAB><TAB>`之后的提示内容，如果提示内容只有唯一一个值，那么将执行自动补全。一般跟`_dothis_completions`这样的函数配合使用，用后者来缩小前者作为集合的size，以方便补全。
- `compgen`
  shell内置的一个命令，用来为`COMPREPLY`提供内容，一般是作为`_dothis_completions`函数的核心语句。用法类似`complete`，关键在于用不同的flag来产生用于提示或补全的内容。
- `COMP_WORDS`
  shell内置的一个变量，用来保存`dothis`之后输入的wordlist。
- `COMP_CWORD`
  shell内置的一个变量，用来表示`dothis`之后输入的last word的索引，或者说用户想要被补全的当前word在`COMP_WORDS`中的位置。

最后，综合以上内容，看一个典型例子：
```shell
_dothis_completions()
{
  COMPREPLY=($(compgen -W "now tomorrow never" "${COMP_WORDS[COMP_CWORD]}"))
}

complete -F _dothis_completions dothis
```
其中核心语句的含义是，用待补全的word的已输入信息`${COMP_WORDS[COMP_CWORD]}`去筛选wordlist`"now tomorrow never"`，筛选结果赋给`COMPREPLY`，用于提示（筛选结果不唯一）或自动补全（筛选结果唯一）。

参考链接：https://iridakos.com/programming/2018/03/01/bash-programmable-completion-tutorial#:~:text=What%20is%20bash%20completion%3F,key%20while%20typing%20a%20command.

### `basename`命令
用于提取一个路径，或者一个字符串中最想要的子串，比如
```shell
# 不指定后缀，输出最后一个'/'之后的内容，这里是gcc-8
basename /usr/bin/gcc-8

# 指定后缀，输出最后一个'/'之后，后缀之前的内容，这里是gcc
basename /usr/bin/gcc-8 -8
```

### `tr`命令
用于字符串转换，比如将一个字符序列中的全部小写字符转换为大写字符：
```shell
cat readme.txt | tr [:lower:] [:upper:]
```

### 
