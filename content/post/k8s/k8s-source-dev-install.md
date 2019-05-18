---
title: "K8s Source Dev Install"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2019-05-10T08:52:24+08:00
draft: true
---

文章简介：介绍 `k8s` 开发环境搭建

<!--more-->

# vagrant env

Linux ubuntu-bionic 4.15.0-47-generic #50-Ubuntu SMP Wed Mar 13 10:44:52 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

# install golang

```bash
wget https://dl.google.com/go/go1.12.5.linux-amd64.tar.gz
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
export GOPATH=$HOME/go
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin:\$GOPATH/bin
```

# install docker

# disabled swap

```bash
swapoff -a
echo "vm.swappiness = 0">> /etc/sysctl.conf
sysctl -p
```

# k8s

## clone k8s source code

```bash
mkdir -p $GOPATH/src/k8s.io
cd $GOPATH/src/k8s.io/
git clone https://github.com/kubernetes/kubernetes.git
```

## startup local cluster

```bash
hack/install-etcd.sh
export PATH="$GOPATH/src/github.com/kubernetes/kubernetes/third_party/etcd:${PATH}"
hack/local-up-cluster.sh [-0]
```

`open another cmd tag`

```bash
export KUBECONFIG=/var/run/kubernetes/admin.kubeconfig
cluster/kubectl.sh

make WHAT=cmd/{\$package_you_want}
```

## see k8s office dev guide

https://github.com/kubernetes/community/blob/master/contributors/devel/development.md

# others

vscode, git

# debug by dlv

# related links

https://farmer-hutao.github.io/k8s-source-code-analysis/prepare/get-code.html
