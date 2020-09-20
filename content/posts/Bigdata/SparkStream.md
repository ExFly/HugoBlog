---
title: "SparkStream等相关产品选型以及Spark安装与简单使用"
author: "Author Name"
cover: "/media/img/Bigdata/spark/spark-logo.png"
tags: ["BigData", "Spark", "Stream"]
date: 2018-07-04T17:36:24+08:00
---

比较SparkStream类似产品如Samza、Storm，介绍Spark和Spark Stream安装和简单使用方法

<!--more-->

# 各产品比较

#### Samza

Samza 是一个分布式的流式数据处理框架（streaming processing），Linkedin 开源的产品， 它是基于 Kafka 消息队列来实现类实时的流式数据处理的。更为准确的说法是，Samza 是通过模块化的形式来使用 Apache Kafka 的，因此可以构架在其他消息队列框架上，但出发点和默认实现是基于 Apache Kafka。

本质上说，Samza 是在消息队列系统上的更高层的抽象，是一种应用流式处理框架在消息队列系统上的一种应用模式的实现。

总的来说，Samza 与 Storm 相比，传输上完全基于 Apache Kafka，集群管理基于 Hadoop YARN，即 Samza 只负责处理这一块具体业务，再加上基于 RocksDB 的状态管理。由于受限于 Kafka 和 YARN，所以它的拓扑结构不够灵活。

#### Storm

Storm 框架与其他大数据解决方案的不同之处，在于它的处理方式。Apcahe Hadoop 本质上来说是一个批处理系统，即目标应用模式是针对离线分析为主。数据被引入Hadoop的分布式文件系统 (HDFS)，并被均匀地分发到各个节点进行处理，HDFS 的数据平衡规则可以参照本文作者发表于IBM的文章《HDFS数据平衡规则及实验介绍》，进行深入了解。当处理完成时，结果数据返回到HDFS，然后可以供处理发起者使用。Storm则支持创建拓扑结构来转换没有终点的数据流。不同于Hadoop作业，这些转换从不会自动停止，它们会持续处理到达的数据，即Storm的流式实时处理方式。

#### Spark Streaming

Spark Streaming 类似于 Apache Storm，用于流式数据的处理。根据其官方文档介绍，Spark Streaming 有高吞吐量和容错能力强这两个特点。Spark Streaming 支持的数据输入源很多，例如：Kafka、Flume、Twitter、ZeroMQ 和简单的 TCP 套接字等等。数据输入后可以用 Spark 的高度抽象原语如：map、reduce、join、window 等进行运算。而结果也能保存在很多地方，如 HDFS，数据库等。另外 Spark Streaming 也能和 MLlib（机器学习）以及 Graphx 完美融合。

在 Spark Streaming中，处理数据的单位是一批而不是单条，而数据采集却是逐条进行的，因此 Spark Streaming系统需要设置间隔使得数据汇总到一定的量后再一并操作，这个间隔就是批处理间隔。批处理间隔（0.2s-2s）是 Spark Streaming 的核心概念和关键参数，它决定了 Spark Streaming 提交作业的频率和数据处理的延迟，同时也影响着数据处理的吞吐量和性能。

#### Kafka Sreeam

Kafka Streams是一个用于处理和分析数据的客户端库。它先把存储在Kafka中的数据进行处理和分析，然后将最终所得的数据结果回写到Kafka或发送到外部系统去。它建立在一些非常重要的流式处理概念之上，例如适当区分事件时间和处理时间、窗口支持，以及应用程序状态的简单（高效）管理。同时，它也基于Kafka中的许多概念，例如通过划分主题进行扩展。此外，由于这个原因，它作为一个轻量级的库可以集成到应用程序中去。这个应用程序可以根据需要独立运行、在应用程序服务器中运行、作为Docker容器，或通过资源管理器（如Mesos）进行操作。

Kafka Sreeam直接解决了流式处理中的很多困难问题:毫秒级延迟的逐个事件处理。有状态的处理，包括分布式连接和聚合。方便的DSL。使用类似DataFlow的模型对无序数据进行窗口化。具有快速故障切换的分布式处理和容错能力。无停机滚动部署。

## 主要比较Spark Stream和Storm和选择

| 比较项 | SparkStream | Storm |
|:---:|:---:|:---:|
| 血统 | UC Berkeley AMP lab | Twitter |
| 开源时间 | 2011.05 | 2011.09 |
| 依赖环境 | Java | Zookeeper Java Python |
| 开发语言 | Scala | Java Clojure |
| 支持语言 | Scala Java Python R | Any |
| 硬盘IO | 少 | 一般 |
| 集群支持 | 超过1000节点 | 好 |
| 吞吐量 | 好 | 较好 |
| 使用公司 | intel 腾讯 淘宝 中移动 Goole | 淘宝 百度 Twitter 雅虎 |
| 适用场景 | 较大数据块&需要高时效性的小批量计算 | 实时小数据块的分析计算 |
| 延时 | 准实时：一次处理一个即将到达的事件 | 实时：处理在一定的时间内（时间间隔可自己设置）在窗口中收到的一批事件 | 
| 容错 | 在批处理级别进行跟踪处理，因此即使发生节点故障等故障，也可以有效地保证每个小批量都能够被精确处理一次 | 每个单独的记录必须在其通过系统时被跟踪，因此Storm仅保证每个记录至少被处理一次，但是从故障中恢复期间允许出现重复。 这意味着可变状态可能不正确地更新了两次 |

1.**处理模型以及延迟**

虽然这两个框架都提供可扩展性(Scalability)和可容错性(Fault Tolerance),但是它们的处理模型从根本上说是不一样的。Storm处理的是每次传入的一个事件，而Spark Streaming是处理某个时间段窗口内的事件流。因此，Storm处理一个事件可以达到亚秒级的延迟，而Spark Streaming则有秒级的延迟。

2.**容错和数据保证**

在容错数据保证方面的权衡方面，Spark Streaming提供了更好的支持容错状态计算。在Storm中，当每条单独的记录通过系统时必须被跟踪，所以Storm能够至少保证每条记录将被处理一次，但是在从错误中恢复过来时候允许出现重复记录，这意味着可变状态可能不正确地被更新两次。而Spark Streaming只需要在批处理级别对记录进行跟踪处理，因此可以有效地保证每条记录将完全被处理一次，即便一个节点发生故障。虽然Storm的 Trident library库也提供了完全一次处理的功能。但是它依赖于事务更新状态，而这个过程是很慢的，并且通常必须由用户实现。

简而言之,如果你需要亚秒级的延迟，Storm是一个不错的选择，而且没有数据丢失。如果你需要有状态的计算，而且要完全保证每个事件只被处理一次，Spark Streaming则更好。Spark Streaming编程逻辑也可能更容易，因为它类似于批处理程序，特别是在你使用批次(尽管是很小的)时。

3.**实现和编程API**

Storm主要是由Clojure语言实现，Spark Streaming是由Scala实现。如果你想看看这两个框架是如何实现的或者你想自定义一些东西你就得记住这一点。Storm是由BackType和 Twitter开发，而Spark Streaming是在UC Berkeley开发的。

Storm提供了Java API，同时也支持其他语言的API。 Spark Streaming支持Scala和Java语言(其实也支持Python)。另外Spark Streaming的一个很棒的特性就是它是在Spark框架上运行的。这样你就可以想使用其他批处理代码一样来写Spark Streaming程序，或者是在Spark中交互查询。这就减少了单独编写流批量处理程序和历史数据处理程序。

4.**生产支持**

Storm已经出现好多年了，而且自从2011年开始就在Twitter内部生产环境中使用，还有其他一些公司。而Spark Streaming是一个新的项目，并且在2013年仅仅被Sharethrough使用(据作者了解)。

Storm是 Hortonworks Hadoop数据平台中流处理的解决方案，而Spark Streaming出现在 MapR的分布式平台和Cloudera的企业数据平台中。除此之外，Databricks是为Spark提供技术支持的公司，包括了Spark Streaming。

5.**集群管理集成**

尽管两个系统都运行在它们自己的集群上，Storm也能运行在Mesos，而Spark Streaming能运行在YARN 和 Mesos上。

```
```

[这里](https://www.cnblogs.com/junneyang/p/8267374.html)总结了Kafka Stream-Spark Streaming-Storm流式计算框架比较选型的相关资料。

这里由更多的相关产品的差异比较资源：

* [Storm介绍](https://www.cnblogs.com/Jack47/p/storm_intro-1.html)
* [Spark Streaming vs. Kafka Stream 哪个更适合你？](http://baijiahao.baidu.com/s?id=1571275750794628&wfr=spider&for=pc)
* [大数据框架对比：Hadoop、Storm、Samza、Spark和Flink
](http://www.infoq.com/cn/articles/hadoop-storm-samza-spark-flink)
* [Spark Streaming与Storm的对比分析](https://blog.csdn.net/kwu_ganymede/article/details/50296831)
* [Storm和Spark Streaming的横向比较](https://www.jianshu.com/p/11f7dec5aa07)
* [Spark Streaming和Storm如何选择？搭建流式实时计算平台，广告日志实时花费](https://www.zhihu.com/question/29092950/answer/131543255)
* [Spark Streaming 新手指南](https://www.ibm.com/developerworks/cn/opensource/os-cn-spark-streaming/index.html)

# Spark 介绍 Spark生态
[Spark官网](https://spark.apache.org/)简单介绍了spark的的优势。

[这里](http://www.10tiao.com/html/357/201708/2247485473/1.html)非常详细了介绍Spark生态、各大厂应用场景、Spark基本原理。

# Spark 和 Spark Stream的安装和使用
## Spark介绍
Spark Streaming 是 Spark Core API 的扩展, 它支持弹性的, 高吞吐的, 容错的实时数据流的处理. 数据可以通过多种数据源获取, 例如 Kafka, Flume, Kinesis 以及 TCP sockets, 也可以通过例如 map, reduce, join, window 等的高级函数组成的复杂算法处理. 最终, 处理后的数据可以输出到文件系统, 数据库以及实时仪表盘中.事实上,你还可以在 data streams（数据流）上使用[机器学习](http://spark.apachecn.org/docs/cn/2.2.0/ml-guide.html)以及[图计算](http://spark.apachecn.org/docs/cn/2.2.0/graphx-programming-guide.html) 算法

![](/media/img/Bigdata/spark/streaming-arch.png)

在内部, 它工作原理如下, Spark Streaming 接收实时输入数据流并将数据切分成多个 batch（批）数据, 然后由 Spark 引擎处理它们以生成最终的 stream of results in batches（分批流结果）.

![](/media/img/Bigdata/spark/streaming-flow.png)
Spark Streaming 提供了一个名为 discretized stream 或 DStream 的高级抽象, 它代表一个连续的数据流. DStream 可以从数据源的输入数据流创建, 例如 Kafka, Flume 以及 Kinesis, 或者在其他 DStream 上进行高层次的操作以创建. 在内部, 一个 DStream 是通过一系列的 RDDs 来表示.

你可以使用 Scala , Java 或者 Python（Spark 1.2 版本后引进）来编写 Spark Streaming 程序. 

[这里是一篇官方编程指南](http://spark.apachecn.org/docs/cn/2.2.0/streaming-programming-guide.html)

## Spark安装

#### 方式1
```bash
wget http://mirror.bit.edu.cn/apache/spark/spark-2.3.1/spark-2.3.1-bin-hadoop2.7.tgz
tar -xzf spark-2.3.1-bin-hadoop2.7.tgz

# 运行一个例子
cd spark-2.3.1-bin-hadoop2.7
./bin/run-example SparkPi
```

#### 方式二
**推荐这种方式**这里总结了自己搭建各种开发环境的就自动化安装脚本。第一次安装会比较麻烦，之后实现一条命令自动安装。需要vagrant&virtual。有一些依赖docker

```bash
git clone https://github.com/ExFly/ComputSciLab.git
cd ComputSciLab
vagrant up
vagrant ssh
cd /vagrant/Java
source install-small.sh
cd /vagrant/Spark
./install.sh
cd /vagrant/.softwenv/spark-2.3.1-bin-hadoop2.7
./bin/run-example SparkPi
```

结果图：

![](/media/img/Bigdata/spark/exfly-spark.png)

#### spark集群
找到一个[中文的文档](http://spark.apachecn.org/docs/cn/2.2.0/index.html),可以看一下，部署很简单

# 总结
如上
