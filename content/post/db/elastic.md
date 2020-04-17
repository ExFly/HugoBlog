---
title: "ElasticSearch"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2020-04-16T08:49:16+08:00
draft: true
---

ElasticSearch 整理一下知识点

<!--more-->

ElasticSearch 是一个分布式搜索引擎，底层使用 Lucene 来实现其核心搜索功能.其核心是全文检索.

## 全文检索

[倒排索引](https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html) + [TF-IDF](https://zhuanlan.zhihu.com/p/31197209)为全文搜索的基石。

## ElasticSearch 诞生的背景

### 大规模数据如何检索

当系统数据量上了 10 亿、100 亿条的时候，我们在做系统架构的时候通常会从以下角度去考虑问题：

1. 用什么数据库好？(mysql、postgres、sybase、oracle、达梦、神通、mongodb、hbase…)
2. 如何解决单点故障；(lvs、F5、A10、Zookeep、MQ)
3. 如何保证数据安全性；(热备、冷备、异地多活)
4. 如何解决检索难题；(数据库代理中间件：mysql-proxy、Cobar、MaxScale 等;)
5. 如何解决统计分析问题；(离线、近实时)

### 传统数据库的应对解决方案

对于关系型数据，我们通常采用以下或类似架构去解决查询瓶颈和写入瓶颈：

1. 通过主从备份解决数据安全性问题；
2. 通过数据库代理中间件心跳监测，解决单点故障问题；
3. 通过代理中间件将查询语句分发到各个 slave 节点进行查询，并汇总结果

### 非关系型数据库的解决方案

对于 Nosql 数据库，基本原理类似：

1. 通过副本备份保证数据安全性；
2. 通过节点竞选机制解决单点问题；
3. 先从配置库检索分片信息，然后将请求分发到各个节点，最后由路由节点合并汇总结果

## Elastic 理论知识

### Elasticsearch vs mysql

Mysql -> database -> table -> rows -> columns
Elasticsearch -> index -> type -> documents -> fields

[elastic 在 7.x 之后将去掉 types](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/removal-of-types.html), 替代方案：index per document type / custom type field

### ES 的 CRUD

[资料](https://blog.csdn.net/zkyfcx/article/details/79998197)

#### 索引新文档

当用户向一个节点提交了一个索引新文档的请求，节点会计算新文档应该加入到哪个分片（shard）中。每个节点都存储有每个分片存储在哪个节点的信息，因此协调节点会将请求发送给对应的节点。注意这个请求会发送给主分片，等主分片完成索引，会并行将请求发送到其所有副本分片，保证每个分片都持有最新数据。

每次写入新文档时，都会先写入内存中，并将这一操作写入一个 translog 文件（transaction log）中，此时如果执行搜索操作，这个新文档还不能被索引到。

ES 会每隔 1 秒时间（这个时间可以修改）进行一次刷新操作（refresh），此时在这 1 秒时间内写入内存的新文档都会被写入一个文件系统缓存（filesystem cache）中，并构成一个分段（segment）。此时这个 segment 里的文档可以被搜索到，但是尚未写入硬盘，即如果此时发生断电，则这些文档可能会丢失。

不断有新的文档写入，则这一过程将不断重复执行。每隔一秒将生成一个新的 segment，而 translog 文件将越来越大。每隔 30 分钟或者 translog 文件变得很大，则执行一次 fsync 操作。此时所有在文件系统缓存中的 segment 将被写入磁盘，而 translog 将被删除（此后会生成新的 translog）。

由上面的流程可以看出，在两次 fsync 操作之间，存储在内存和文件系统缓存中的文档是不安全的，一旦出现断电这些文档就会丢失。所以 ES 引入了 translog 来记录两次 fsync 之间所有的操作，这样机器从故障中恢复或者重新启动，ES 便可以根据 translog 进行还原。

此外，由于每一秒就会生成一个新的 segment，很快将会有大量的 segment。对于一个分片进行查询请求，将会轮流查询分片中的所有 segment，这将降低搜索的效率。因此 ES 会自动启动合并 segment 的工作，将一部分相似大小的 segment 合并成一个新的大 segment。合并的过程实际上是创建了一个新的 segment，当新 segment 被写入磁盘，所有被合并的旧 segment 被清除。

#### 更新（Update）和删除（Delete）文档

ES 的索引是不能修改的，因此更新和删除操作并不是直接在原索引上直接执行。每一个磁盘上的 segment 都会维护一个 del 文件，用来记录被删除的文件。每当用户提出一个删除请求，文档并没有被真正删除，索引也没有发生改变，而是在 del 文件中标记该文档已被删除。因此，被删除的文档依然可以被检索到，只是在返回检索结果时被过滤掉了。每次在启动 segment 合并工作时，那些被标记为删除的文档才会被真正删除。

更新文档会首先查找原文档，得到该文档的版本号。然后将修改后的文档写入内存，此过程与写入一个新文档相同。同时，旧版本文档被标记为删除，同理，该文档可以被搜索到，只是最终被过滤掉。

#### 读操作（Read）：查询过程

##### 查询阶段

当一个节点接收到一个搜索请求，则这个节点就变成了协调节点。第一步是广播请求到索引中每一个节点的分片拷贝。 查询请求可以被某个主分片或某个副本分片处理，协调节点将在之后的请求中轮询所有的分片拷贝来分摊负载。

每个分片将会在本地构建一个优先级队列。如果客户端要求返回结果排序中从第 from 名开始的数量为 size 的结果集，则每个节点都需要生成一个 from+size 大小的结果集，因此优先级队列的大小也是 from+size。分片仅会返回一个轻量级的结果给协调节点，包含结果集中的每一个文档的 ID 和进行排序所需要的信息。

协调节点会将所有分片的结果汇总，并进行全局排序，得到最终的查询排序结果。此时查询阶段结束。

##### 取回阶段

查询过程得到的是一个排序结果，标记出哪些文档是符合搜索要求的，此时仍然需要获取这些文档返回客户端。

协调节点会确定实际需要返回的文档，并向含有该文档的分片发送 get 请求；分片获取文档返回给协调节点；协调节点将结果返回给客户端。

## 分布式一致性原理

[资源，有源码分析](https://zhuanlan.zhihu.com/p/34830403)

### ES 集群构成

node.master node.data 两两组合成不同的节点

### 节点发现

### Master 选举

[多数派原则](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-quorums.html) 选主

## references

- [elasticsearch cheatsheet](https://elasticsearch-cheatsheet.jolicode.com/)
- [Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/master/index.html)
