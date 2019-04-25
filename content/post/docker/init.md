---
title: "docker 源码阅读：开发环境搭建 + docker源码分析"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["docker", "container"]
date: 2019-04-23T12:48:28+08:00
---

文章简介：介绍 docker 开发环境搭建

<!--more-->

docker 官方的贡献引导: [moby/docs/contributing/README.md](https://github.com/moby/moby/blob/master/docs/contributing/README.md)

# 步骤

[moby/docs/contributing/set-up-dev-env.md](https://github.com/moby/moby/blob/master/docs/contributing/set-up-dev-env.md)完整的描述了开发环境如何搭建

### 开发环境搭建

- 因为网络问题，需要尽快的提速安装 dev 环境，需要配置 Dockerfile 中的配置 APT_MIRROR=mirrors.163.com
- 运行`make --just-print BIND_DIR=. shell`简单的看一下 make 都做了那些事情
  ![make --just-print BIND_DIR=. shell](/media/img/docker/install-dev-env/docker-make-just-print-shell.png)

- 运行`make BIND_DIR=. shell`开始安装开发环境

### 编译

上一节已经进入 docker 中，可以开始编译 dockerd

- `hack/make.sh binary`
- `make install`copy binary to container's `/usr/local/bin/`
- `dockerd -D &`

### 日常工作流

- 修改代码
- `hack/make.sh binary install-binary`

# 调试

## 调试 makefile

`make --just-print BIND_DIR=. shell`

## 调试 shell

`bash -x hack/make.sh binary`
