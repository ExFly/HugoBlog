---
title: "报告实现"
date: 2020-09-21T08:28:04+08:00
---

文章简介：报告系统设计

<!--more-->

需求是无外网环境下，汇总当前系统的运行状态，需要绘制大量图表到 word/pdf。

实现语言: golang

## 如何更加方便的管理报告模版

主要思路：

报告模版使用 markdown+golang/text/template 渲染，图表使用 datauri 直接内嵌入 markdown。markdown 经过 ast 解析后翻译成对应的元素，渲染成 report.Report 中间接口。report.Report 接口作为中间层，下边实现 pdf/word 渲染实现。

最终达成的目标：样式由 report.Report 的实现决定，上层 markdown 关注内容。

> github.com/yuin/goldmark

## 报表中大量图表如何渲染

使用前端技术绘制图表，可在浏览器中直接获得结果

渲染服务：控制浏览器打开网页，并指定截图某区域.

有一个关键设计: chromedp 控制浏览器，会没有基础的 post请求方式，需要通过控制浏览器执行 js, 对网页进行控制（创建 dom ，并画出图）

渲染服务无状态，需要管理 chrome headless。对外提供 openapi

golang

> https://github.com/browserless/chrome/issues/52
> github.com/chromedp/chromedp

## 如何保存最终生成的报告

markdown 直接存储到数据库，优点不需要做大量小文件优化
