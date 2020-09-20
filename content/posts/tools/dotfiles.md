---
title: "整理了一套 dotfiles 自用"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["tools", "dev"]
date: 2019-09-28T17:20:14+08:00
---

文章简介：自己的 `dotfiles` 供自己使用，几乎一键换机

<!--more-->

# 简介

使用方法

```
git clone https://github.com/exfly/dotfiles.git ~/.dotfiles
cd ~/.dotfiles
make preinstall-arch
make bootstrap
make install
```

## 备注

对于所有的 `*.zsh` 都会加载到 `.zshrc` 中，所有的 `*.symlink` 都会对应的在`$HOME`生成一个软连接 `.*`, 文件夹也可以直接使用这种方式进行加载。

比如 `zsh/zshrc.symlink` 会生成软连接 `$HOME/.zshrc`,具体如何工作可以看一下 `script/bootstrap`

# references

- [my dotfiles](https://github.com/exfly/dotfiles)
