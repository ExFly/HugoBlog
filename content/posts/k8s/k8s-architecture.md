---
title: "K8s Architecture"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2019-05-10T08:52:13+08:00
draft: true
---

文章简介：k8s 架构描述

<!--more-->

https://kubernetes.io/docs/concepts/

# 架构图

![from https://x-team.com/blog/introduction-kubernetes-architecture/](/media/img/k8s/kubernates.png)

![content/media/img/k8s/k8s-high-level-component-arch.jpeg](/media/img/k8s/k8s-high-level-component-arch.jpeg)

# 组件

etcd + docker/rkt + Kubernetes + kubelet + kube-proxy + kube-apiserver + kube-controller-manager + kube-scheduler

## 基本概念

copied from [here](https://www.kubernetes.org.cn/doc-19)

- Cluster : 集群是指由 Kubernetes 使用一系列的物理机、虚拟机和其他基础资源来运行你的应用程序。
- Node : 一个 node 就是一个运行着 Kubernetes 的物理机或虚拟机，并且 pod 可以在其上面被调度。.
- Pod : 一个 pod 对应一个由相关容器和卷组成的容器组 （了解 Pod 详情）
- Label : 一个 label 是一个被附加到资源上的键/值对，譬如附加到一个 Pod 上，为它传递一个用户自定的并且可识别的属性.Label 还可以被应用来组织和选择子网中的资源（了解 Label 详情）
- selector: 是一个通过匹配 labels 来定义资源之间关系得表达式，例如为一个负载均衡的 service 指定所目标 Pod.（了解 selector 详情）
  Replication Controller : replication controller 是为了保证一定数量被指定的 Pod 的复制品在任何时间都能正常工作.它不仅允许复制的系统易于扩展，还会处理当 pod 在机器在重启或发生故障的时候再次创建一个（了解 Replication Controller 详情）
- Service : 一个 service 定义了访问 pod 的方式，就像单个固定的 IP 地址和与其相对应的 DNS 名之间的关系。（了解 Service 详情）
- Volume: 一个 volume 是一个目录，可能会被容器作为未见系统的一部分来访问。（了解 Volume 详情）
- Kubernetes volume 构建在 Docker Volumes 之上,并且支持添加和配置 volume 目录或者其他存储设备。
- Secret : Secret 存储了敏感数据，例如能允许容器接收请求的权限令牌。
- Name : 用户为 Kubernetes 中资源定义的名字
- Namespace : Namespace 好比一个资源名字的前缀。它帮助不同的项目、团队或是客户可以共享 cluster,例如防止相互独立的团队间出现命名冲突
- Annotation : 相对于 label 来说可以容纳更大的键值对，它对我们来说可能是不可读的数据，只是为了存储不可识别的辅助数据，尤其是一些被工具或系统扩展用来操作的数据

# related links

- [Kubernetes 设计架构-完整文档](https://www.kubernetes.org.cn/kubernetes%E8%AE%BE%E8%AE%A1%E6%9E%B6%E6%9E%84)
