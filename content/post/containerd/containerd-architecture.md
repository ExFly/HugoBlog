---
title: "Containerd Architecture"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["containerd"]
date: 2019-05-26T18:50:31+08:00
draft: true
---

文章简介：balabala

<!--more-->

# containerd

containerd 启动 grpc server 向外部服务

具体的服务实现类在此：

- content-servece services/content/contentserver/contentserver.go

# ctr

```
sudo ctr --namespace docker tasks

sudo ctr content fetch docker.io/crosbymichael/runc:latest
sudo ctr install docker.io/crosbymichael/runc:latest
sudo ctr content fetch docker.io/library/redis:alpine
sudo ctr run --rm  docker.io/library/redis:alpine redis-demo-c
sudo ls /run/containerd/io.containerd.runtime.v1.linux/default/redis-demo-c/
```

# references

- [containerd-architecture](https://github.com/containerd/containerd/blob/master/design/architecture.md)
