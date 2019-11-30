---
title: "读写操作到底有多慢, 操作系统相关一些参数"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["os"]
date: 2019-11-30T13:11:38+08:00
---

文章简介：Latency numbers every programmer should know

<!--more-->

```
L1 cache reference ......................... 0.5 ns
Branch mispredict ............................ 5 ns
L2 cache reference ........................... 7 ns
Mutex lock/unlock ........................... 25 ns
Main memory reference ...................... 100 ns
Compress 1K bytes with Zippy ............. 3,000 ns  =   3 µs
Send 2K bytes over 1 Gbps network ....... 20,000 ns  =  20 µs
SSD random read ........................ 150,000 ns  = 150 µs
Read 1 MB sequentially from memory ..... 250,000 ns  = 250 µs
Round trip within same datacenter ...... 500,000 ns  = 0.5 ms
Read 1 MB sequentially from SSD* ..... 1,000,000 ns  =   1 ms
Disk seek ........................... 10,000,000 ns  =  10 ms
Read 1 MB sequentially from disk .... 20,000,000 ns  =  20 ms
Send packet CA->Netherlands->CA .... 150,000,000 ns  = 150 ms
```
[src](https://gist.github.com/exfly/03434b3b064593ddc78ec911ba2f3e7c#file-latency-markdown)
