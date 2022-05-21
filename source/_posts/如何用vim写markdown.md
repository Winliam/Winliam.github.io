---
title: 如何用vim写markdown
date: 2019-07-08 18:25:26
categories: 
  - - 生活记录
    - 电脑设置
tags:
  - vim
  - markdown
---

# 安装neovim
>sudo apt-get install neovim  
>neovim  
>:checkhealth

# 安装vim插件管理器vim-plug
>sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs 
https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'

链接不上的话，需要：<br>
>sudo nvim /etc/hosts<br>

添加一行：<br>

>199.232.96.133 raw.githubusercontent.com<br>

其中的ip地址来自于*https://githubusercontent.com.ipaddress.com/raw.githubusercontent.com*

# 创建neovim配置文件init.vim

> mkdir .config/nvim  
> touch .config/nvim/init.vim

# 修改init.vim以添加插件
目录，markdown，preview一共三个暂时（修改的内容直接看文件）,
中间: 
- 遇到了自动折叠问题：修改配置文件解决  
- 又遇到了无法预览的问题，看github[作者回复](https://github.com/iamcco/markdown-preview.nvim/issues/120)解决
