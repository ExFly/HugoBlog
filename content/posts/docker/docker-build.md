---
title: "Docker 源码阅读: Docker Build"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["docker", "源码"]
date: 2019-05-05T08:41:00+08:00
---

文章简介：docker build 源码阅读

<!--more-->

# 阅读源码顺序

- api/server/router/build/build_routes.go#postBuild
- api/server/backend/build/backend.go#Build
- builder/builder-next/builder.go#Build
- builder/dockerfile/builder.go#Build
- builder/dockerfile/builder.go#dispatchDockerfileWithCancellation
- builder/dockerfile/evaluator.go#dispatch，在这里，有所有命令的执行方式，可以仔细研究一下

在`dispatch`中有 dockerfile 支持的指令，如`RUN`等。以 `RUN` 为例，`dockerd` 会读取 `CLI` 发来的 `dockerfile`，解析后。如果为 `RUN`，则会启动一个新的 `container`,然后在容器中执行，执行结束后将当前层 `commit`，继续执行下一个指令.

# 其他

- 从 `v18.09`开始，`docker build` 依赖于 `buildkit`
