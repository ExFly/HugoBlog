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

## 挑战

1. 成倍的 API 数量

- 引入网络延迟
- CAP 理论，处理跨多个服务的事务复杂
- 调试分布式系统十分复杂
- 服务雪崩
- 大量请求堆积，故障恢复慢
- 微服务技术选型
- API 版本管理混乱
- TCC、事务消息队列

## 需要做的事

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

## 需要深入的技术栈

分布式系统、DevOps、基础架构即代码(IaC)、不同类型的数据库、前端组件化和复合化、单元测试、全自动发布、迭代、小版本发布计划、测试工具、多版本管理

# 分布式一致性与共识算法

[Consensus from wiki](<https://en.wikipedia.org/wiki/Consensus_(computer_science)>), 总结下来一致性就是，在分布式系统中，在给定的一系列操作，即使系统内部出错，最终整个系统对外提供的数据都是可靠的。在协调一致性的过程中，对于一个 Proposal 整个系统达成共识，共识算法起着很重要的作用。

## CAP

加州伯克利大学的教授 Eric Brewer 论文[Brewer’s Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.67.6951&rep=rep1&type=pdf) 阐述了 CAP 理论 [CAP from wiki](https://en.wikipedia.org/wiki/CAP_theorem), 论文中提出的观点：在异步的网络模型中，所有的节点由于没有时钟仅仅能根据接收到的消息作出判断，这时完全不能同时保证一致性(Consistency)、可用性(Availability)和分区容错性(Partition tolerance)，每一个系统只能在这三种特性中选择两种。
由于网络有一定的延迟，并不能做到强一致性，所以大部分时候采用最终一致性的方式，容忍一定时间内数据不一致，在一定的时间内系统内部各节点可以在有限的时间内解决冲突，使数据恢复准确的状态。

## 拜占庭将军问题

[拜占庭将军问题论文](https://web.archive.org/web/20170205142845/http://lamport.azurewebsites.net/pubs/byz.pdf)

拜占庭将军问题是对分布式系统容错的最高要求，然而这不是日常工作中使用的大多数分布式系统中会面对的问题，我们遇到更多的还是节点故障宕机或者不响应等情况，这就大大简化了系统对容错的要求

## FLP

FLP 不可能定理是分布式系统领域最重要的定理之一，它给出了一个非常重要的结论：**在网络可靠并且存在节点失效的异步模型系统中，不存在一个可以解决一致性问题的确定性算法。**

> In this paper, we show the surprising result that no completely asynchronous consensus protocol can tolerate even a single unannounced process death. We do not consider Byzantine failures, and we assume that the message system is reliable it delivers all messages correctly and exactly once.

[相关论文](https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf)

# 共识算法

## paxos, raft

包括 Paxos、Raft

[The Raft Consensus Algorithm](https://raft.github.io/)
[raft 工作原理动图](http://thesecretlivesofdata.com/raft/)

Paxos 算法难以理解、难以实现，难道什么程度呢？在 raft 的论文中有提及。比如 zookeeper 的 zab 就是在 paxos 的基础上进行设计的。每一种 Paxos 的实现，都需要重新设计实现一套算法。而 raft 相对难度降低很多。
基于 raft 的 etcd，consul 等。

## POW(Proof-of-Work)

无论是 Paxos 还是 Raft 其实都只能解决非拜占庭将军容错的一致性问题，不能够应对分布式网络中出现的极端情况，但是这在传统的分布式系统都不是什么问题，无论是分布式数据库还是消息队列集群，它们内部的节点并不会故意的发送错误信息，在类似系统中，最常见的问题就是节点失去响应或者失效，所以它们在这种前提下是有效可行的，也是充分的。

[工作量证明](https://en.wikipedia.org/wiki/Proof-of-work_system)是一个用于阻止拒绝服务攻击和类似垃圾邮件等服务错误问题的协议

## POS(Proof-of-Stake)

权益证明是区块链网络中的使用的另一种共识算法，在基于权益证明的密码货币中，下一个区块的选择是根据不同节点的股份和时间进行随机选择的。

## DPOS(Delegated Proof-of-Stake)

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
- [分布式一致性与共识算法](https://draveness.me/consensus)
