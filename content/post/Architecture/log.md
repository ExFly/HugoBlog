---
title: "Log"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["architecture"]
date: 2019-09-16T09:03:53+08:00
---

文章简介：log

<!--more-->

在多个系统相互协作的分布式系统中，为了能够保证异步系统的协作是否工作，需要一套日志系统保证系统正常的工作。

# 工具

- 应用内日志，比如开一个新的 `log table`，其中记录关键元数据，以及一些返回结果。应用内日志记录的内容未来可以作为数据 migration 的有效工具，保证没有成功的操作可以在未来重试，保证系统的数据的完整
- [opentracing](https://opentracing.io) 跟踪请求链路，方便 debug 分布式系统。（具体如何做，之后补充 TODO: flag）
- elk 系列
- nginx 日志
