---
title: "分布式数据的最终一致性"
author: "Author Name"
cover: "/media/img/icon/logo43.svg"
tags: ["Architecture", "Distributed"]
date: 2018-06-28T16:34:49+08:00
---

单体应用，需要借助分库分表、复制技术、读写分离提高服务并发访问量。微服务为代表的分布式系统，其高并发和微服务事务一致性该如何保证？

<!--more-->

# 简介
由于自己刚刚接触，自己理解的也不深。在这里，把我整理的一些资料汇总下来。

# 微服务架构
微服务架构将单应用放在多个相互独立的服务，这个每个服务能够持续独立的开发和部署，难题是数据该如何存储？

## 多个应用使用同一数据库
![](/media/img/architecture/EventuallyConsistent/1server1db.jpeg)
传统的单体应用一般采用的是数据库提供的事务一致性，通过数据库提供的提交以及回滚机制来保证相关操作的ACID，这些操作要么同时成功，要么同时失败。各个服务看到数据库中的数据是一致的，同时数据库的操作也是相互隔离的，最后数据也是在数据库中持久存储的。这样的架构不具备横向扩展能力，服务之间的耦合程度也比较高，会存在单点故障。

## 典型微服务架构
![](/media/img/architecture/EventuallyConsistent/micservice.jpeg)
在微服务架构中， 有一个database per service的模式， 这个模式就是每一个服务一个数据库。 这样可以保证微服务独立开发，独立演进，独立部署， 独立团队。

由于一个应用是由一组相互协作的微服务所组成，在分布式环境下由于各个服务访问的数据是相互分离的， 服务之间不能靠数据库来保证事务一致性。 这就需要在应用层面提供一个协调机制，来保证一组事务执行要么成功，要么失败。

# CAP定律
一个分布式系统最多只能同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance）这三项中的两项。

通过CAP理论，我们知道无法同时满足一致性、可用性和分区容错性这三个特性，那要舍弃哪个呢？

CA without P：如果不要求P（不允许分区），则C（强一致性）和A（可用性）是可以保证的。但其实分区不是你想不想的问题，而是始终会存在，因此CA的系统更多的是允许分区后各子系统依然保持CA。

CP without A：如果不要求A（可用），相当于每个请求都需要在Server之间强一致，而P（分区）会导致同步时间无限延长，如此CP也是可以保证的。很多传统的数据库分布式事务都属于这种模式。

AP wihtout C：要高可用并允许分区，则需放弃一致性。一旦分区发生，节点之间可能会失去联系，为了高可用，每个节点只能用本地数据提供服务，而这样会导致全局数据的不一致性。现在众多的NoSQL都属于此类。

对于多数大型互联网应用的场景，主机众多、部署分散，而且现在的集群规模越来越大，所以节点故障、网络故障是常态，而且要保证服务可用性达到N个9，即保证P和A，舍弃C（退而求其次保证最终一致性）。虽然某些地方会影响客户体验，但没达到造成用户流程的严重程度。

对于涉及到钱财这样不能有一丝让步的场景，C必须保证。网络发生故障宁可停止服务，这是保证CA，舍弃P。貌似这几年国内银行业发生了不下10起事故，但影响面不大，报到也不多，广大群众知道的少。还有一种是保证CP，舍弃A。例如网络故障事只读不写。

## 常用的解决方法
[这里](http://www.infoq.com/cn/articles/solution-of-distributed-system-transaction-consistency)总结了一些分布式数据一致性的解决方法。分布式事务保证强一致性，但为了保证数据的一致性，放弃了一些系统性能。另一种保证最终一致性，放弃了时时数据的一致性，但处理效率最好。

[这里有一些例子](https://github.com/JoeCao/JoeCao.github.io/issues/5)如何解决的。

## BASE
[这里](https://www.cnblogs.com/birdstudio/p/7373057.html)实验了一个基于BASE协议的最终一致性demo。注意，这里使用到了Kafka，需要自己在本地开Kafka服务。

## 其他资料
* 书：大规模分布式存储系统：原理解析与架构实现
* 书：微服务设计
* [分布式事务？No, 最终一致性](https://zhuanlan.zhihu.com/p/25933039)
* [分布式事务实践 -花钱的，作为目录使用](https://coding.imooc.com/class/chapter/237.html)
* [多研究些架构，少谈些框架（3）-- 微服务和事件驱动](https://github.com/JoeCao/JoeCao.github.io/issues/5)
* [消息中间件（一）分布式系统事务一致性解决方案大对比，谁最好使？](https://blog.csdn.net/lovesomnus/article/details/51785108)
* [Saga分布式事务解决方案与实践](https://servicecomb.incubator.apache.org/cn/docs/distributed-transactions-saga-implementation/)
* [解决业务代码里的分布式事务一致性问题](https://zhuanlan.zhihu.com/p/25346771)
* [分布式事务实践](https://coding.imooc.com/class/chapter/237.html#Anchor)
* **实战**[基于Kafka消息驱动最终一致事务（二）](https://www.cnblogs.com/birdstudio/p/7373057.html)
