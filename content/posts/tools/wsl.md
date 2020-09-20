---
title: "WSL(windows subsystem for linux) win子系统"
author: "Author Name"
cover: "/media/img/wsl/ubuntu.icon.png"
tags: ["工具", "wsl"]
date: 2018-12-08T16:00:59+08:00
---

win 子系统安装与 cmder+zsh 开发环境搭建

<!--more-->

# 说在前面

**这里只展示可以做到什么程度，具体怎么做，网上教程很多，后边会贴出自己感觉比较好的网址**

# 大致操作思路

先在 windows 下[打开 win subsystem for linux 功能](https://zhuanlan.zhihu.com/p/34133795)，之后去 win store 中下载对应的 linux 发行版.之后就是打开对应的 linux 发行版的 bash、配置 zsh 了，详细步骤见[这里](https://zhuanlan.zhihu.com/p/34152045)。具体 zsh 怎么折腾，可以看一下[这里](https://github.com/robbyrussell/oh-my-zsh/wiki)

贴一张自己配置之后，使用 zsh 和 tmux 之后的截图
![wsl](/media/img/wsl/wsl.png)

给我的体验是，基本可以满足大部分日常开发工作

## 其他

- win 下的 CDEF 盘被挂载到`/mnt`下，为了方便使用，可以将他们`ln -s /mnt/d $HOME/windir`，这样方便自己使用
- 因为之前安装过 vscode，可以直接使用`code filename` 打开系统中的文件
- 类似 jdk 这种需要在 subsystem 中重新安装才可以在 wsl 中使用
- 图中使用的 Cmder 是非常远古的版本，所以请忽略 cmd 之前的框

# 链接

- [Windows10 终端优化方案：Ubuntu 子系统+cmder+oh-my-zsh
  ](https://zhuanlan.zhihu.com/p/34152045)
