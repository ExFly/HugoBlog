---
title: "消息队列"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["mq", "architecture", "distributed"]
date: 2019-07-29T08:04:30+08:00
---

文章简介：消息队列中需要解决的问题

<!--more-->

网络通信可能会包含，成功、失败以及超时三种情况

# 消息投递语义

最多一次（At-Most Once）、最少一次（At-Least Once）以及恰好一次（Exactly Once）

**最多一次**在 TCP/UDP 传输层协议就是保证最多一次消息投递，消息的发送者只会尝试发送该消息一次，并不会关心该消息是否得到了远程节点的响应。
**最少一次**引入超时重试的机制。同时引入新的问题，消息重复。
**恰好一次**从理论上来说，在分布式系统中想要解决消息重复的问题是不可能的，很多消息服务提供了正好一次的 QoS 其实是在接收端进行了去重。

# 投递顺序

由于一些网络的问题，消息在投递时可能会出现顺序不一致性的情况，在网络条件非常不稳定时，我们就可能会遇到接收方处理消息的顺序和生产者投递的不一致；消费者就需要对顺序不一致的消息进行处理，常见的两种方式就是使用**序列号**或者**状态机**。

## 序列号

用阻塞的方式保证序列号的递增或者忽略部分『过期』的消息。

## 状态机

虽然消息投递的顺序是乱序的，但是资源最终还是通过状态机达到了我们想要的正确状态，不会出现不一致的问题。

# 协议

AMQP： StormMQ、RabbitMQ， 支持最多一次和最少一次的投递语义，当我们选择最少一次时，需要幂等或者重入机制保证消息重复不会出现问题。

MQTT

# reference

- [分布式事务的实现原理](https://draveness.me/distributed-transaction-principle)
- [分布式系统与消息的投递](https://draveness.me/message-delivery)
- [浅谈数据库并发控制 - 锁和 MVCC](https://draveness.me/database-concurrency-control)
- [消息队列设计的精髓基本都藏在本文里了](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2650993240&idx=1&sn=bb2c943ee7aeabc3b5962056534d36b5#rd)
- [消息队列设计精要](https://tech.meituan.com/2016/07/01/mq-design.html)
- [消息队列设计精要](https://tech.meituan.com/2016/07/01/mq-design.html)
