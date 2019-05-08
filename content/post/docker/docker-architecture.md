---
title: "Docker 源码阅读: Docker架构介绍"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["docker", "源码"]
date: 2019-04-22T21:35:46+08:00
---

文章简介：`docker`基本架构，以及在 moby 开源项目相关的一些组件之间如何协同工作的

<!--more-->

# Docker?

## 我们可以使用 Docker 做什么

快速，一致地交付您的应用程序

Docker 允许开发人员使用提供应用程序和服务的本地容器在标准化环境中工作，从而简化了开发生命周期。 容器非常适合持续集成和持续交付（CI / CD）工作流程。

# docker 全局架构

![docker架构图](/media/img/docker/engine-components-flow.png)

CLI 使用 Docker REST API 通过脚本或直接 CLI 命令控制 Docker 守护程序或与 Docker 守护程序交互。 许多其他 Docker 应用程序使用底层 API 和 CLI。

Docker daemon 创建和管理 Docker 对象，例如 images，containers，networks 和 volumes。

![docker架构图2](/media/img/docker/architecture.svg)

## The Docker daemon

Docker daemon（dockerd）监听 Docker API 请求并管理 Docker 对象，例如 images，containers，networks 和 volumes。 Docker daemon 还可以与其他守护程序通信以管理 Docker 服务。

> Note: 例如 dockerd 通过 rest 接口与 containnerd 进行交互，containerd 与 runc 通过 grpc 进行通信

## The Docker client

Docker 客户端（docker）是许多 Docker 用户与 Docker 交互的主要方式。 当您使用诸如 docker run 之类的命令时，客户端会将这些命令发送到 dockerd，dockerd 将其执行。 docker (CLI) 命令调用 REST 风格的 Docker API。 Docker 客户端可以与多个守护进程通信。

> Note: docker 可以通过配置环境变量 `DOCKER_HOST`或者修改配置变量，或者命令行参数的方式连接 dockerd

## Docker registries

Docker registry 存储 Docker 镜像。 Docker Hub 是任何人都可以使用的公共注册中心，Docker 配置为默认在 Docker Hub 上查找 images。 您甚至可以运行自己的私人 registry。 如果您使用 Docker Datacenter（DDC），它包含 Docker Trusted Registry（DTR）。

> Note: 可以通过配置不同的 Docker registry 作为 images 下载源，比如公司内部搭建 registry 私服，在平时 CI/CD 中提高工作速度。

## Docker objects

使用 Docker 时，您正在创建和使用 images，containers，networks 和 volumes，plugins 和其他对象。 本节简要介绍其中一些对象。

### IMAGES

images 是一个只读模板，其中包含有关创建 Docker 容器的说明。 通常，images 基于另一个 images，并带有一些额外的自定义。 例如，您可以构建基于 ubuntu images 的 images，但安装 Apache Web 服务器和应用程序，以及运行应用程序所需的配置详细信息。

您可以创建自己的 images，也可以只使用其他人创建的 images 并在 registries 中发布。 要构建自己的 images，可以使用简单的语法创建 Dockerfile，以定义创建 images 并运行 images 所需的步骤。 Dockerfile 中的每条指令都在 images 中创建一个 layer。 更改 Dockerfile 并重建 images 时，仅重建已更改的那些层。 与其他虚拟化技术相比，这是使 images 如此轻量，小巧和快速的部分原因。

> Note: 比较详细的工作原理可以看下文 `Union file systems`部分

### CONTAINERS

CONTAINERS 是 images 的可运行实例。 您可以使用 Docker API 或 CLI 创建，启动，停止，移动或删除容器。 您可以将容器连接到一个或多个网络，将存储连接到它，甚至可以根据其当前状态创建新 images。

默认情况下，CONTAINERS 与其他 CONTAINERS 及其主机相对隔离。 您可以控制 CONTAINERS 的网络，存储或其他基础子系统与其他 CONTAINERS 或主机的隔离程度。

CONTAINERS 由其 images 以及您在创建或启动时为其提供的任何配置选项定义。 删除 containers 后，对其状态的任何未存储在持久存储中的更改都将消失。

### SERVICES

services 允许您跨多个 Docker 守护程序扩展容器，这些守护程序一起作为具有多个管理器和工作程序的群组一起工作。 swarm 的每个成员都是 Docker 守护程序，守护进程都使用 Docker API 进行通信。 服务允许您定义所需的状态，例如在任何给定时间必须可用的服务的副本数。 默认情况下，服务在所有工作节点之间进行负载平衡。 对于消费者来说，Docker 服务似乎是一个单独的应用程序。 Docker Engine 支持 Docker 1.12 及更高版本中的 swarm 模式。

# Docker 底层技术

Docker 是用 `Go` 编写的，它利用 Linux 内核的几个功能来提供其功能。使用到的内核特性包括 namespaces、cgroups、Union file systems

## namespaces

Docker 使用称为 namespaces 的技术来提供称为容器的隔离工作空间。 运行容器时，Docker 会为该容器创建一组 namespaces。

这些 namespaces 提供了一层隔离。 容器的每个方面都在一个单独的命名空间中运行，其访问权限仅限于该 namespaces。

Docker Engine 在 Linux 上使用以下命名空间：

- pid 命名空间：进程隔离（PID：进程 ID）。
- net 命名空间：管理网络接口（NET：Networking）。
- ipc 名称空间：管理对 IPC 资源的访问（IPC：进程间通信）。
- mnt 名称空间：管理文件系统挂载点（MNT：Mount）。
- uts 命名空间：隔离内核和版本标识符。 （悉尼科技大学：Unix 分时系统）。

## cgroups

Linux 上的 Docker Engine 还依赖于另一种称为控制组（cgroups）的技术。 cgroup 将应用程序限制为特定的资源集。 cgroups 允许 Docker Engine 将可用的硬件资源共享给容器，并可选择强制执行限制和约束。 例如，您可以限制特定容器的可用内存。

## Union file systems

联合文件系统或 UnionFS 是通过创建 layers 来操作的文件系统，使它们非常轻量和快速。 Docker Engine 使用 UnionFS 为容器提供构建块。 Docker Engine 可以使用多种 UnionFS 变体，包括 AUFS，btrfs，vfs 和 DeviceMapper。

> Note: Docker 默认为 overylay2

## Container format

Docker Engine 将 namespaces，cgroups 和 UnionFS 组合成一个称为容器格式的包装器。 默认容器格式是 libcontainer。 将来，Docker 可以通过与 BSD Jails 或 Solaris Zones 等技术集成来支持其他容器格式。

> Note: 如上部分翻译自 [docker overview](https://docs.docker.com/engine/docker-overview/)，同时添加自己本人理解。

# moby 等代码的依赖介绍，相关调用方式，架构

![docker-components](/media/img/docker/docker-components.png)

Docker 从一个单一的软件转移到一组独立的组件和项目。

## Docker 如何运行容器？

- Docker 引擎创建 images，
- 把 images 传递给 containerd，
- containerd 调用 containerd-shim，
- containerd-shim 使用 runC 来运行 container，
- containerd-shim 允许运行时（在本例中为 runC）在启动容器后退出

## 这个模型的两个主要好处是

- deamon 运行较少的容器
- 能够在不破坏正在运行的容器的情况下重启或升级引擎

# related link

- [moby](https://github.com/moby/moby)
- [libnetwork](https://github.com/docker/libnetwork)
- [containerd](https://github.com/containerd/containerd)
- [runc](https://github.com/opencontainers/runc)

- [Visualizing Docker Containers and Images](http://merrigrove.blogspot.com/2015/10/visualizing-docker-containers-and-images.html)
