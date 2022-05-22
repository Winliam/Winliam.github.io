---
title: coredump文件的使用
categories:
  - - 工作技能
    - 操作系统
date: 2021-05-04 10:36:38
tags:
  - coredump
  - gdb
  - debug
---
## 简介
程序崩溃之后，一般会生成一个coredump文件。

跟可重定位目标文件和可执行目标文件一样，这个文件也是一个elf文件，是程序崩溃瞬间进程栈和寄存器状态的快照。

可以使用gdb加载这个文件，查看崩溃瞬间的程序运行状态。

## coredump开关
```shell
#查看允许生成的文件大小上限，coresize=0意味着不允许产生, unlimited表示无限制
ulimit -a

#设置权限
ulimit -c unlimited
```

## coredump文件路径
```shell
#查看路径
cat /proc/sys/kernel/core_pattern

#设置路径
##临时修改
echo ‘/var/log/%e.core.%p’ > /proc/sys/kernel/core_pattern

##永久修改
/sbin/sysctl -w kernel.core_pattern=/var/log/%e.core.%p
```

## gdb加载方式
```shell
gdb [exe] [core_file]
gdb mainboard /apollo/data/core/core_mainboard.5852
```

## gdb常用命令
```shell
bt			#查看调用栈
where			#同上
frame stk_num		#切换函数栈
print var_name		#查看变量值
```
