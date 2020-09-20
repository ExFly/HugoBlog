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
predicates.FitPredicate implement: `NoDiskConflict`

#### "Prioritizing"

"Prioritizing"：调用 PrioritizeNodes()方法；

#### "Selecting host"

"Selecting host"：调用 selectHost()方法。

#### List-Watch

#### Informer

### 抢占调度

pkg/scheduler/scheduler.go#preempt

# 总结

kube-scheduler 工作流程

对给定的 pod，先列举所有可调度的 nodes，根据资源是否充足初步过滤；使用优先级算法对 nodes 进行计算排序；最终选择分数最高的一个 node

总的工作流程图：

![kube-scheduler workflow](/media/img/k8s/scheduler/kube-scheduler-workflow.png) referenced from [here](https://farmer-hutao.github.io/k8s-source-code-analysis/core/scheduler/summarize.html)

# Reference

- [kube-scheduler workflow](https://farmer-hutao.github.io/k8s-source-code-analysis/core/scheduler/summarize.html)
