---
title: nvidia-smi突然跪了！
categories:
  - - 生活记录
    - 电脑设置
date: 2021-07-26 17:51:09
tags:
  - Nvidia
  - 驱动失效
---
## 解决过程
看着代码唱着歌，CUDA就突然啥啥啥初始化失败了。

nvidia-smi一看，又是一个新报错Failed to initialize NVML: Driver/library version mismatch，闹心。

整半天，大概意思是作为内核模块的nvidia驱动程序版本和libnvidia-compute-460这个库的版本不一致。

两者一般都应该是安装驱动的时候指定的版本（for me is 460.84），但是后者竟然通过ubuntu的后台更新程序自动更新了（460.91），所以造成了这个问题。

最后，更新了驱动程序，关闭了自动更新，reboot，告一段落。

## 用到的命令：
> lsmod | grep nvidia* 
> 显示和nvidia有关的内核模块有哪些
> 
> modinfo nvidiaxxx | grep version 
> 显示这些内核模块的版本
> 
> dpkg --list | grep nvidia 
> 显示libnvidia-compute这个库的版本
>
> dmesg | tail -4 
> nvidia-smi的报错详情，可以看到版本差异
> dmesg | grep nvidia 
> nvidia-smi的执行过程详情
>
> cat /var/log/apt/history.log | grep -a -C 10 nvidia 
> libnvidia-compute的更新记录，可以看到是unattended-upgrade造成的更新
> 
> cat /etc/apt/apt.conf.d/20auto-upgrades
> 自动更新的配置文件，修改即可关闭自动更新[detail](https://unix.stackexchange.com/questions/342663/how-is-unattended-upgrades-started-and-how-can-i-modify-its-schedule)
>
> sudo apt install nvidia-driver-460
> 更新驱动程序

## 参考链接
1. [https://stackoverflow.com/questions/62250491/nvml-driver-library-mismatch-after-libnvidia-compute-update](https://stackoverflow.com/questions/62250491/nvml-driver-library-mismatch-after-libnvidia-compute-update)


