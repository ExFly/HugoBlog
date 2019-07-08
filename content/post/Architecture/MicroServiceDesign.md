---
title: "高可用系统设计问题、微服务设计笔记 脑图"
author: "Exfly"
cover: "/media/img/architecture/icon.jpg"
tags: ["Architecture", "微服务", "Distributed"]
date: 2019-07-07T22:50:19+08:00
---

高可用系统设计，有哪些问题需要解决

<!--more-->

# 什么是高可用系统

[关于高可用的系统](https://coolshell.cn/articles/17459.html)

# 高可用系统需要解决的问题

1. 可扩展：水平扩展、垂直扩展。 通过冗余部署，避免单点故障。

- 隔离：避免单一业务占用全部资源。避免业务之间的相互影响 2. 机房隔离避免单点故障。
- 解耦：降低维护成本，降低耦合风险。减少依赖，减少相互间的影响。
- 限流：滑动窗口计数法、漏桶算法、令牌桶算法等算法。遇到突发流量时，保证系统稳定。
- 降级：紧急情况下释放非核心功能的资源。牺牲非核心业务，保证核心业务的高可用。
- 熔断：异常情况超出阈值进入熔断状态，快速失败。减少不稳定的外部依赖对核心服务的影响。
- 系统监控：对 CPU 利用率，load，内存，带宽，系统调用量，应用错误量，PV，UV 和业务量进行监控
- 自动化测试：通过完善的测试，减少发布引起的故障。
- 灰度发布(+回滚)：灰度发布是速度与安全性作为妥协，能够有效减少发布故障。
- 异步消息系统

# 《微服务设计》脑图

脑图如下：
![脑图](https://github.com/ExFly/CsLearning/raw/master/NoteBookForDevelop/%E4%B9%A6%E7%AC%94%E8%AE%B0/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E8%AE%BE%E8%AE%A1/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E8%AE%BE%E8%AE%A1.png)

- [脑图地址](https://github.com/ExFly/CsLearning/tree/master/NoteBookForDevelop/%E4%B9%A6%E7%AC%94%E8%AE%B0/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E8%AE%BE%E8%AE%A1)

# references

- [后端架构师技术图谱](https://github.com/xingshaocheng/architect-awesome/blob/master/README.md)
- [hystrix](https://github.com/Netflix/Hystrix/wiki)
- [bilibili/kratos](https://github.com/bilibili/kratos)

- [Go Microservices blog series](https://callistaenterprise.se/blogg/teknik/2017/02/17/go-blog-series-part1/)
- [microservices-in-golang](https://ewanvalentine.io/microservices-in-golang-part-1/)
- [shippy-demo](https://github.com/EwanValentine/shippy/tree/master/user-service)
- [20 个好用的 Go 语言微服务开发框架](https://zhuanlan.zhihu.com/p/52778237)
- [micro/micro](https://github.com/micro/micro/)
- [micro/go-micro](https://github.com/micro/go-micro)
- [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem)
- [grpc-example](https://github.com/gogo/grpc-example)