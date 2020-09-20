---
title: "Go Issue"
date: 2020-06-27T07:33:02+08:00
---

记录 golang 相关很难调试的 bug

<!--more-->

## bugs

### SIGSEGV during C.getaddrinfo

> https://github.com/golang/go/issues/30310

glibc 的 bug 导致 go 中 net 模块静态编译后的可执行文件 getaddrinfo 的时候 pinic，无法启动服务
解决办法: 临时使用 go 自己的 net 实现 `-tags netgo`, 或者 已经 build 的程序 `GODEBUG=netdns=go binery` 加环境变量使用 go 的 net 实现
