---
title: "K8s Apiserver"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2019-05-24T08:17:57+08:00
draft: true
---

文章简介：kube-apiserver

<!--more-->

# 源码阅读流程

cmd/kube-apiserver/app/server.go#NewAPIServerCommand
cmd/kube-apiserver/app/server.go#Run
cmd/kube-apiserver/app/server.go#CreateServerChain

vendor/k8s.io/apiextensions-apiserver/pkg/apiserver/customresource_discovery_controller.go#Run

cmd/kube-apiserver/app/server.go#CreateKubeAPIServer

apiserver 的真正 url Hanlder 在 genericapiserver.GenericAPIServer.Handler
cmd/kube-apiserver/app/aggregator.go#createAggregatorServer 真正开始初始化 genericapiserver.GenericAPIServer.Handler apiServicesToRegister 也有初始化 handler
