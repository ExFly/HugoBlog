---
title: "Linux epoll"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["epoll"]
date: 2020-04-06T12:54:50+08:00
---

epoll 高性能 network io 模型

<!--more-->

> 由于太多 blog 已经讲 epoll，再此引用看的blog，以及项目。并根据自己的理解，实现出一个 epoll 实现的 server。

> TODO: 需要跟一步设计实现跨平台的序列化反序列化方案，进一步实现应用层需求。

# epoll

高性能 异步io 模型，在 Linux 是 epoll，mac kqueue，windows IOCP.

App: nginx, redis

## references

- [EPOLL的LINUX内核工作机制](http://oenhan.com/epoll-linux-kernel)
- https://blog.lucode.net/linux/epoll-tutorial.html
- https://github.com/millken/c-example/blob/master/epoll-example.c
- https://www.suchprogramming.com/epoll-in-3-easy-steps/
- https://medium.com/@copyconstruct/the-method-to-epolls-madness-d9d2d6378642
- https://github.com/angrave/SystemProgramming/wiki/Networking,-Part-7:-Nonblocking-I-O,-select(),-and-epoll
- https://github.com/angrave/SystemProgramming/wiki
- https://jvns.ca/blog/2017/06/03/async-io-on-linux--select--poll--and-epoll
- [simple http server base on epoll](https://github.com/hongliuliao/ehttp/)
- https://github.com/cloudwu/cstring

# [camel](https://github.com/exfly/camel)

使用 linux epoll 实现的 socket server.
