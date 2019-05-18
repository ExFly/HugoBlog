---
title: "K8s Scheduler"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2019-05-10T09:26:35+08:00
draft: true
---

文章简介：balabala

<!--more-->

[office scheduler info](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-scheduling/scheduler.md)

# code

`pkg/scheduler/scheduler.go#scheduleOne` 真正的执行逻辑

# 组件

## scheduler

调度会 binding 到 apiserver，异步的等待 pods 启动`pkg/scheduler/scheduler.go#bind`

默认 pod 调度算法：`pkg/scheduler/core/generic_scheduler.go#genericScheduler`

### 如何调度

#### "Computing predicates"

"Computing predicates"：调用 findNodesThatFit()方法；

#### "Prioritizing"

"Prioritizing"：调用 PrioritizeNodes()方法；

#### "Selecting host"

"Selecting host"：调用 selectHost()方法。

## List-Watch

## Informer
