---
title: "Newdb"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2019-06-29T15:13:28+08:00
---

文章简介：简单介绍最近自己最近设计实现 newdb 的工作

<!--more-->

项目地址 [anydemo/newdb](https://github.com/anydemo/newdb)

最近一段时间花了很大的力气学习数据库的知识，设计实现了简单的 RDBMS, 收获比较丰富

开始先从数据库底层的数据持久化开始。再此设计的数据库主要是固定类型大小的关系型数据库。一个 table 中有固定大小的一定数量的数据列，每一个 tuple 的大小是固定不变的，也就是每一个 page 的可以容纳的 tuple 是一定的，这样很容易在记录某一个 tuple 的位置、以及其在文件中的位置。在这过程中收获最大的部分就是对于 Marshal 于 Unmarshal 的理解更进一步，知道数据如何在数据库中是如何存储的，以及事务的本质。

最后通过一个 Sequences Scan 的例子，手写一个物理执行计划，执行一次全表扫描。

未来还有很多的内容可以做，暂时告一段落，最近抽时间沉淀一下，思考一下未来，回头有时间了再回头继续加新的内容。毕竟[issues](https://github.com/anydemo/newdb/issues)积攒了太多。
