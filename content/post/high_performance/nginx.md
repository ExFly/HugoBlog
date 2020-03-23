---
title: "Nginx源码分析"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2020-03-22T09:19:46+08:00
draft: true
---

文章简介：balabala

<!--more-->

## 关键代码/函数

ngx_process_events_and_timers
ngx_event_core_module
ngx_events_module 解决惊群问题

ngx_trylock_accept_mutex

ngx_http_upstream_t
ngx_http_upstream_handler
ngx_http_upstream_send_request_handler

## references

- [Debugging NGINX](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/)
- [看懂启动及进程工作原理](https://www.jianshu.com/p/b9e1b6b46a2a)
- [taobao nginx平台初探](https://tengine.taobao.org/book/index.html)