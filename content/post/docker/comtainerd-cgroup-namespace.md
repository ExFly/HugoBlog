---
title: "Comtainerd Cgroup Namespace"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["cgroup", "namespace"]
date: 2019-09-17T09:07:19+08:00
---

文章简介：Docker Cgroup , Namespace , union fs 运行机制

<!--more-->

# Namespaces

命名空间 (namespaces) 是 Linux 为我们提供的用于分离进程树、网络接口、挂载点以及进程间通信等资源的方法.

Linux 的命名空间机制提供了以下七种不同的命名空间，包括 CLONE_NEWCGROUP、CLONE_NEWIPC、CLONE_NEWNET、CLONE_NEWNS、CLONE_NEWPID、CLONE_NEWUSER 和 CLONE_NEWUTS(分别对应 Cgroup, IPC, Network, Mount, PID, User, UTS)，通过这七个选项我们能在创建新的进程时设置新进程应该在哪些资源上与宿主机器进行隔离。

> the Network namespace encapsulates system resources related to networking such as network interfaces (e.g wlan0, eth0), route tables etc, the Mount namespace encapsulates files and directories in the system, PID contains process IDs and so on. So two instances of a Network namespace A and B (corresponding to two boxes of the same type in our analogy) can contain different resources - maybe A contains wlan0 while B contains eth0 and a different route table copy -- [related](http://ifeanyi.co/posts/linux-namespaces-part-1/)

```sh
$ ls -l /proc/$$/ns
total 0
lrwxrwxrwx 1 vagrant vagrant 0 Sep 19 00:46 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 vagrant vagrant 0 Sep 19 00:46 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 vagrant vagrant 0 Sep 19 00:46 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 vagrant vagrant 0 Sep 19 00:46 net -> 'net:[4026531992]'
lrwxrwxrwx 1 vagrant vagrant 0 Sep 19 00:46 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 vagrant vagrant 0 Sep 19 00:46 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 vagrant vagrant 0 Sep 19 00:46 user -> 'user:[4026531837]'
lrwxrwxrwx 1 vagrant vagrant 0 Sep 19 00:46 uts -> 'uts:[4026531838]'
```

```sh
$ sudo unshare -u bash
$ ls -l /proc/$$/ns
total 0
lrwxrwxrwx 1 root root 0 Sep 19 01:00 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Sep 19 01:00 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Sep 19 01:00 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Sep 19 01:00 net -> 'net:[4026531992]'
lrwxrwxrwx 1 root root 0 Sep 19 01:00 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Sep 19 01:00 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Sep 19 01:00 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Sep 19 01:00 uts -> 'uts:[4026532239]'
```

`unshare` 会在一个新的 namespace 执行新的程序，flag `-u` 指定新的 `UTS` namespace, `sudo unshare -u bash`即是在新的`UTS`命名空间执行 `bash`

linux 以默认的 namespace 启动新的程序，除非明确的指定

## user namespace

> P$ Q$ 不同的字母代表不同的 user namespace

```sh
P$ whoami
vagrant

P$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant),977(docker)

P$ unshare -U bash
# Enter a new shell that runs within a nested user namespace, the shell is bash which is your cmd shell

C$ id
uid=65534(nobody) gid=65534(nobody) groups=65534(nobody)

C$ ls -l
```

### `/proc/$pid/uid_map`

> the map file `/proc/$pid/uid_map` returns a mapping from UIDs in the user namespace to which the process pid belongs， [user_namespace linux man page](http://man7.org/linux/man-pages/man7/user_namespaces.7.html)

每一行的格式为 `$fromID $toID $length`，

```sh
C$ echo $$
8683 # 新的 PID

P$ echo $$
7898 # 原始 PID

C$ cat /proc/8683/uid_map

C$ cat /proc/7898/uid_map
         0          0 4294967295
```

```sh
P$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant),977(docker)

P$ ip link add type veth
RTNETLINK answers: Operation not permitted

# 在一个不同的username 和 net namespace中尝试
P$ unshare -nU bash

C$ ip link add type veth
RTNETLINK answers: Operation not permitted

C$ echo $$
9078

P$ echo "0 1000 1" > /proc/9078/uid_map

C$ id
uid=0(root) gid=65534(nobody) groups=65534(nobody)

C$ ip link add type veth
# Success!

P$ echo deny > /proc/9078/setgroups

P$ echo "0 1000 1" > /proc/9078/gid_map

C$ id
uid=0(root) gid=0(root) groups=0(root),65534(nobody)
```

### [mount namespace](http://man7.org/linux/man-pages/man7/mount_namespaces.7.html)

```sh
cat /proc/$$/mounts

proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
sys /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
dev /dev devtmpfs rw,nosuid,relatime,size=496792k,nr_inodes=124198,mode=755 0 0
run /run tmpfs rw,nosuid,nodev,relatime,mode=755 0 0
/dev/sda2 / btrfs rw,relatime,space_cache,subvolid=5,subvol=/ 0 0
securityfs /sys/kernel/security securityfs rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /dev/shm tmpfs rw,nosuid,nodev 0 0
devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000 0 0
tmpfs /sys/fs/cgroup tmpfs ro,nosuid,nodev,noexec,mode=755 0 0
cgroup2 /sys/fs/cgroup/unified cgroup2 rw,nosuid,nodev,noexec,relatime,nsdelegate 0 0
cgroup /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,xattr,name=systemd 0 0
pstore /sys/fs/pstore pstore rw,nosuid,nodev,noexec,relatime 0 0
bpf /sys/fs/bpf bpf rw,nosuid,nodev,noexec,relatime,mode=700 0 0
cgroup /sys/fs/cgroup/cpuset cgroup rw,nosuid,nodev,noexec,relatime,cpuset 0 0
cgroup /sys/fs/cgroup/pids cgroup rw,nosuid,nodev,noexec,relatime,pids 0 0
cgroup /sys/fs/cgroup/perf_event cgroup rw,nosuid,nodev,noexec,relatime,perf_event 0 0
cgroup /sys/fs/cgroup/net_cls,net_prio cgroup rw,nosuid,nodev,noexec,relatime,net_cls,net_prio 0 0
cgroup /sys/fs/cgroup/rdma cgroup rw,nosuid,nodev,noexec,relatime,rdma 0 0
cgroup /sys/fs/cgroup/freezer cgroup rw,nosuid,nodev,noexec,relatime,freezer 0 0
cgroup /sys/fs/cgroup/cpu,cpuacct cgroup rw,nosuid,nodev,noexec,relatime,cpu,cpuacct 0 0
cgroup /sys/fs/cgroup/devices cgroup rw,nosuid,nodev,noexec,relatime,devices 0 0
cgroup /sys/fs/cgroup/blkio cgroup rw,nosuid,nodev,noexec,relatime,blkio 0 0
cgroup /sys/fs/cgroup/memory cgroup rw,nosuid,nodev,noexec,relatime,memory 0 0
cgroup /sys/fs/cgroup/hugetlb cgroup rw,nosuid,nodev,noexec,relatime,hugetlb 0 0
hugetlbfs /dev/hugepages hugetlbfs rw,relatime,pagesize=2M 0 0
configfs /sys/kernel/config configfs rw,nosuid,nodev,noexec,relatime 0 0
systemd-1 /proc/sys/fs/binfmt_misc autofs rw,relatime,fd=46,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=10711 0 0
mqueue /dev/mqueue mqueue rw,nosuid,nodev,noexec,relatime 0 0
debugfs /sys/kernel/debug debugfs rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /tmp tmpfs rw,nosuid,nodev 0 0
tmpfs /run/user/1000 tmpfs rw,nosuid,nodev,relatime,size=100836k,mode=700,uid=1000,gid=1000 0 0
vagrant /vagrant vboxsf rw,relatime 0 0
```

[mount point](http://www.linfo.org/mount_point.html)

```sh
$ unshare -m bash
$ cat /proc/$$/mounts
/dev/sda2 / btrfs rw,relatime,space_cache,subvolid=5,subvol=/ 0 0
dev /dev devtmpfs rw,nosuid,relatime,size=496792k,nr_inodes=124198,mode=755 0 0
tmpfs /dev/shm tmpfs rw,nosuid,nodev 0 0
devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000 0 0
hugetlbfs /dev/hugepages hugetlbfs rw,relatime,pagesize=2M 0 0
mqueue /dev/mqueue mqueue rw,nosuid,nodev,noexec,relatime 0 0
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
systemd-1 /proc/sys/fs/binfmt_misc autofs rw,relatime,fd=46,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=10711 0 0
sys /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
securityfs /sys/kernel/security securityfs rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /sys/fs/cgroup tmpfs ro,nosuid,nodev,noexec,mode=755 0 0
cgroup2 /sys/fs/cgroup/unified cgroup2 rw,nosuid,nodev,noexec,relatime,nsdelegate 0 0
cgroup /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,xattr,name=systemd 0 0
cgroup /sys/fs/cgroup/cpuset cgroup rw,nosuid,nodev,noexec,relatime,cpuset 0 0
cgroup /sys/fs/cgroup/pids cgroup rw,nosuid,nodev,noexec,relatime,pids 0 0
cgroup /sys/fs/cgroup/perf_event cgroup rw,nosuid,nodev,noexec,relatime,perf_event 0 0
cgroup /sys/fs/cgroup/net_cls,net_prio cgroup rw,nosuid,nodev,noexec,relatime,net_cls,net_prio 0 0
cgroup /sys/fs/cgroup/rdma cgroup rw,nosuid,nodev,noexec,relatime,rdma 0 0
cgroup /sys/fs/cgroup/freezer cgroup rw,nosuid,nodev,noexec,relatime,freezer 0 0
cgroup /sys/fs/cgroup/cpu,cpuacct cgroup rw,nosuid,nodev,noexec,relatime,cpu,cpuacct 0 0
cgroup /sys/fs/cgroup/devices cgroup rw,nosuid,nodev,noexec,relatime,devices 0 0
cgroup /sys/fs/cgroup/blkio cgroup rw,nosuid,nodev,noexec,relatime,blkio 0 0
cgroup /sys/fs/cgroup/memory cgroup rw,nosuid,nodev,noexec,relatime,memory 0 0
cgroup /sys/fs/cgroup/hugetlb cgroup rw,nosuid,nodev,noexec,relatime,hugetlb 0 0
pstore /sys/fs/pstore pstore rw,nosuid,nodev,noexec,relatime 0 0
bpf /sys/fs/bpf bpf rw,nosuid,nodev,noexec,relatime,mode=700 0 0
configfs /sys/kernel/config configfs rw,nosuid,nodev,noexec,relatime 0 0
debugfs /sys/kernel/debug debugfs rw,nosuid,nodev,noexec,relatime 0 0
tracefs /sys/kernel/debug/tracing tracefs rw,nosuid,nodev,noexec,relatime 0 0
run /run tmpfs rw,nosuid,nodev,relatime,mode=755 0 0
tmpfs /run/user/1000 tmpfs rw,nosuid,nodev,relatime,size=100836k,mode=700,uid=1000,gid=1000 0 0
tmpfs /tmp tmpfs rw,nosuid,nodev 0 0
vagrant /vagrant vboxsf rw,relatime 0 0
```

说明 `unshare -m bash` 并没有按照预期在新的 mount namespace 中运行

所以为了验证`mount namespace`, 我们需要做如下一些事情:

1. 创建命令所需的依赖项和系统文件的副本
1. 创建一个新的 `mount namespace`
1. 将新挂载命名空间中的根文件系统替换为由我们的系统文件副本组成的根文件系统。
1. 在新的安装名称空间内执行程序

#### 进行实验

```sh
$ wget http://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86_64/alpine-minirootfs-3.10.1-x86_64.tar.gz
$ mkdir rootfs
$ tar -xzf alpine-minirootfs-3.10.1-x86_64.tar.gz -C rootfs
$ ls rootfs

$ ls rootfs/{mnt,dev,proc,home,sys}
```

##### Pivot root

```
$ unshare -m bash
$ mount --bind rootfs rootfs
$ cd rootfs
$ mkdir put_old
$ pivot_root . put_old
$ cd /
# We should now have our new root. e.g if we:
$ ls proc
# proc is empty
# And the old root is now in put_old
$ ls put_old
bin   dev  home        lib    lost+found  mnt  proc  run   srv  tmp  var
boot  etc  initrd.img  lib64  media       opt  root  sbin  sys  usr  vmlinuz

$ /bin/busybox umount -l put_old # 卸载旧的文件系统
$ mount -t proc proc proc/
```

到这里，已经创建了一个隔离的 rootfs

## 进程

```sh
$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 01:17 ?        00:00:00 /sbin/init
root         2     0  0 01:17 ?        00:00:00 [kthreadd]
root         3     2  0 01:17 ?        00:00:00 [rcu_gp]
root         4     2  0 01:17 ?        00:00:00 [rcu_par_gp]
root         5     2  0 01:17 ?        00:00:00 [kworker/0:0-events]
root         6     2  0 01:17 ?        00:00:00 [kworker/0:0H-kblockd]
root         7     2  0 01:17 ?        00:00:00 [kworker/u4:0-btrfs-endio-write]
root         8     2  0 01:17 ?        00:00:00 [mm_percpu_wq]
```

有两个进程很特殊，pid={1,2}, 这两个进程都是被 Linux 中的上帝进程 idle 创建出来的

- pid=1 的 `/sbin/init`, 执行内核的一部分初始化工作和系统配置，也会创建一些类似 getty 的注册进程
- pid=2 的`kthreadd`, 管理和调度其他的内核进程

## 网络

在默认情况下，每一个容器在创建时都会创建一对虚拟网卡，两个虚拟网卡组成了数据的通道，其中一个会放在创建的容器中，会加入到名为 docker0 网桥中。我们可以使用如下的命令来查看当前网桥的接口：

```sh
$ brctl show
bridge name     bridge id               STP enabled     interfaces
br-43ee5ed53f84         8000.024297630322       no
docker0         8000.0242bf0282f8       no
```

docker0 会为每一个容器分配一个新的 IP 地址并将 docker0 的 IP 地址设置为默认的网关。网桥 docker0 通过 iptables 中的配置与宿主机器上的网卡相连，所有符合条件的请求都会通过 iptables 转发到 docker0 并由网桥分发给对应的机器。

```sh
$ iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL

Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere
```

#### ip

```sh
$ ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:d4:b6:c8 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
    link/ether 08:00:27:b7:f5:14 brd ff:ff:ff:ff:ff:ff

$ ip netns add coke
$ ip netns list
coke
```

`ip netns list` 只显示命名的 netns，初始 netns 是非命名 netns。并且每一个命名 netns 都会在`/var/run/netns`创建同名文件，这个文件可以让进程切换到此 netns

`C$` 代表在一个子命名空间中执行

```sh
$ ip netns exec coke bash

C$ ip link list
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

C$ ping 127.0.0.1
connect: Network is unreachable

C$ ip link set dev lo up

C$ ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

C$ ping 127.0.0.1
# OK
```

现在在此 netns 中只可以与同 netns 的进程进行沟通(localhost),我们可以尝试与 init netns 的程序进行沟通

### Veth Devices

```sh
$ ip link add veth0 type veth peer name veth1 # # Create a veth pair (veth0 <=> veth1)
$ ip link set veth1 netns coke # # Move the veth1 end to the new namespace
C$ ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
7: veth1@if5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether ee:16:0c:23:f3:af brd ff:ff:ff:ff:ff:ff link-netnsid 0

$ ip addr add 10.1.1.1/24 dev veth0
```

现在 veth1 设备已经可以在两个 netns 中看到，为了是他们都可以工作，我们需要给它们两个`ip addresses` 和让`interface up`

```sh
$ ip addr add 10.1.1.1/24 dev veth0 # In the initial namespace
$ ip link set dev veth0 up

C$ ip addr add 10.1.1.2/24 dev veth1 # # In the coke namespace
C$ ip link set dev veth1 up
C$ ip addr show veth1
4: veth1@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 1a:e7:9f:4e:d4:db brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.1.1.2/24 scope global veth1
       valid_lft forever preferred_lft forever
    inet6 fe80::18e7:9fff:fe4e:d4db/64 scope link
       valid_lft forever preferred_lft forever
```

现在 `veth0` 和 `veth1` 已经 `up` 和赋予了 ip address `10.1.1.1` `10.1.1.2`

```sh
$ ping -I veth0 10.1.1.2

C$ ping 10.1.1.1
```

### netlink

### libnetwork

## chroot

https://www.ibm.com/developerworks/cn/linux/l-cn-chroot/index.html

# CGroups

https://www.ibm.com/developerworks/cn/linux/1506_cgroup/index.html

# UnionFS

docker export \$(docker create busybox) | tar -C rootfs -xvf -
AUFS overlay2

查看系统是否支持相应的文件系统

```sh
$ uname -a
Linux archlinux 5.1.15-arch1-1-ARCH #1 SMP PREEMPT Tue Jun 25 04:49:39 UTC 2019 x86_64 GNU/Linux

$ grep btrfs /proc/filesystems
       btrfs
```

有一些系统这种方式找不到对应的系统，但是同样支持此文件系统，比如 `archlinux` 发行版，[Overlay_filesystem](https://wiki.archlinux.org/index.php/Overlay_filesystem), [overlay-docker-doc](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)

```sh
$ mkdir b0 b1 b2 upper work merged
$ mount -t overlay overlay -o lowerdir=./b0:./b1:./b2,upperdir=./upper,workdir=./work ./merged
```

overlay2 将 lowerdir、upperdir、workdir 联合挂载，形成最终的 merged 挂载点，其中 lowerdir 是镜像只读层，upperdir 是容器可读可写层，workdir 是执行涉及修改 lowerdir 执行 copy_up 操作的中转层（例如，upperdir 中不存在，需要从 lowerdir 中进行复制，该过程暂未详细了解，遇到了再分析）

```sh
$ tree
.
├── b0
├── b1
├── b2
├── merged
├── README.md
├── upper
└── work
    ├── index
    └── work
```

获得的文件系统是有层次的，当前的层次关系为：

```
/upper
/b0
/b1
/b2
```

```sh
$ echo '192.168.0.1' > b0/ip.txt
$ tree
.
├── b0
│   └── ip.txt
├── b1
├── b2
├── merged
│   └── ip.txt
├── README.md
├── upper
└── work
    ├── index
    └── work

$ cat merged/ip.txt
192.168.0.1

$ echo '192.168.0.2' > merged/ip.txt
$ tree
.
├── b0
│   └── ip.txt
├── b1
├── b2
├── merged
│   └── ip.txt
├── README.md
├── upper
│   └── ip.txt
└── work
    ├── index
    └── work

$ cat upper/ip.txt
192.168.0.2

$ echo '192.168.0.3' > upper/ip.txt
$ cat merged/ip.txt
192.168.0.3

$ rm -rf merged/ip.txt
$ tree
.
├── b0
│   └── ip.txt
├── b1
│   └── ip.txt
├── b2
├── merged
├── README.md
├── upper
│   └── ip.txt
└── work
    ├── index
    └── work

$ ls upper/ip.txt
c--------- 1 root root 0, 0 Sep 18 04:35 upper/ip.txt
```

# references

- [container-lab](https://github.com/lanything/container-lab)
- https://draveness.me/docker
- 调试网络 `docker run -it --net container:vibrant_blackburn nicolaka/netshoot`
- https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt
- [`overlayfs`的一些限制/兼容性问题](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)
- [linux-namespace-isolate-programing](http://ifeanyi.co/posts/linux-namespaces-part-1/) 有一个例子程序
- [Namespaces in operation](https://lwn.net/Articles/531114/)
- [cgroup/linux-man-page](http://man7.org/linux/man-pages/man7/cgroups.7.html)
- [user-namespace/linum-man-page](http://man7.org/linux/man-pages/man7/user_namespaces.7.html)
- [mount namespaces](http://man7.org/linux/man-pages/man7/mount_namespaces.7.html)
- [proc special filesystem](https://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html)
- [management ip netns](http://man7.org/linux/man-pages/man8/ip-netns.8.html)
- [ip command cheatsheet](https://access.redhat.com/sites/default/files/attachments/rh_ip_command_cheatsheet_1214_jcs_print.pdf)
- [veth](http://man7.org/linux/man-pages/man4/veth.4.html)
- [Netlink-doc](http://www.infradead.org/~tgr/libnl/doc/core.html#_introduction)
- [man netlink](http://man7.org/linux/man-pages/man7/netlink.7.html)

- [A deep dive into Linux namespaces](http://ifeanyi.co/posts/linux-namespaces-part-1/)

- [100个容器周边项目，点亮你的容器集群技能树](https://juejin.im/entry/5b558e3751882561b75a5865)
