---
title: "T"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2019-04-23T21:35:46+08:00
draft: true
---

文章简介：balabala

<!--more-->

# docker 全局架构

![docker架构图](/media/img/docker/docker-architecture.jpg)

## 模块

DockerClient DockerDaemon DockerRegistry Graph Driver libcontainer DockerContainer

libcontainer 涉及大量 linux 内核特性，包括 namespaces,cgroups,capabilities

### 各部分关联

通信方式 tcp://host:port unix://path_to_socket fd://socketfd

[from here](https://www.huweihuang.com/kubernetes-notes/docker/docker-architecture.html)

- 用户是使用 Docker Client 与 Docker Daemon 建立通信，并发送请求给后者。
- Docker Daemon 作为 Docker 架构中的主体部分，首先提供 Server 的功能使其可以接受 Docker Client 的请求；
- Engine 执行 Docker 内部的一系列工作，每一项工作都是以一个 Job 的形式的存在。
- Job 的运行过程中，当需要容器镜像时，则从 Docker Registry 中下载镜像，并通过镜像管理驱动 graphdriver 将下载镜像以 Graph 的形式存储；
- 当需要为 Docker 创建网络环境时，通过网络管理驱动 networkdriver 创建并配置 Docker 容器网络环境；
- 当需要限制 Docker 容器运行资源或执行用户指令等操作时，则通过 execdriver 来完成。
- libcontainer 是一项独立的容器管理包，networkdriver 以及 execdriver 都是通过 libcontainer 来实现具体对容器进行的操作。
