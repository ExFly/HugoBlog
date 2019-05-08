---
title: "Docker 源码阅读: Docker Volume"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["docker", "源码"]
date: 2019-05-05T08:43:53+08:00
---

文章简介：描述 docker volume 的原理

<!--more-->

docker 存储基于 UnionFS 实现，所有容器存储是将多个 layout 层通过类似 aufs 或者 overlay2 将多个层联合到一起，mount 成新的目录，在通过 namespaces 将新的文件目录 chroot 进入新启动的进程，达到文件隔离的目的。

- 对于读请求，由于写实复制技术，会直接读取底层文件
- 对于写请求，会先将文件复制到读写层，然后进行修改文件
- 对于删除，没有真正删除文件，只是讲文件标记删除，没有真正删除下层文件。

# 命令

- `docker volume create volume-for-test`可以创建新的 volume

- `docker volume ls` 查看新创建的 volume

- `docker volume inspect volume-for-test`

- `docker run -d --name devtest --mount source=volume-for-test,target=/app alpine /bin/sh`

- `docker inspect <container_id>| grap Mounts` 可以看到刚刚 mount 进去的 volume `/var/lib/docker/volumes/volume-for-test/_data`

# related links

- [Manage data in Docker](https://docs.docker.com/storage/)
- [Use volumes](https://docs.docker.com/storage/volumes/)
- [docker 存储基础](https://li-sen.github.io/2018/12/23/docker%E5%AD%98%E5%82%A8%E5%9F%BA%E7%A1%80/)
