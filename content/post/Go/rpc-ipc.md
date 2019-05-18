---
title: "Go 启动多个程序，及 IPC 和 RPC 交互例子,以及 Gracefully Shutdown"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["go", "IPC", "RPC"]
date: 2019-05-05T18:54:46+08:00
---

文章简介：Go 程序启动 RPC 子进程，通过 pipe 进行交互，以及通过 RPC 交互

<!--more-->

代码见 [exfly/go-ipc](https://github.com/exfly/cslab/tree/master/Code/Go/go-ipc)

# 引言

为什么要写这篇文章。最近看了一些 Docker 源码。dockerd 的架构长[这个样子](/post/docker/docker-architecture/)。 一条 docker 命令的执行，比如`docker run`，是先由 containerd 执行，containerd 也同样不是真正运行容器，他会将执行请求发给 runc，有 runc 真正去执行。dockerd、containderd、runc 分别是三个可执行文件，他们是通过 管道（IPC）以及 rest、RPC 进行交互的。

为了展示三者交互方式是如何进行的，这里写一个简单的 demo 来解释。

# 正文

## RPC

首先说一下 RPC。Go 标准库便有`net/rpc`，写一个 Go 的 rpc 很简单：

```go
package main
import (
	"log"
	"net"
	"net/http"
	"net/rpc"
)

type Task []string
type Todo {ID string}
func (t Task) Get(id string, reply *Todo) error {
    *reply=Todo{ID:id}
    return nil
}

func main() {
	task := new(Task)
	// Publish the receivers methods
	err := rpc.Register(task)
	if err != nil {
		log.Fatal("Format of service Task isn't correct. ", err)
	}
	// Register a HTTP handler
	rpc.HandleHTTP()
	listener, e := net.Listen("unix", "rpc.sock")
	if e != nil {
		log.Fatal("Listen error: ", e)
	}
    err = http.Serve(listener, nil)
    if err != nil {
        log.Fatal("Error serving: ", err)
    }
}
```

```go
// client
client, err := rpc.DialHTTP("unix", "rpc.sock")
reply := TODO{}
err = client.Call("Task.Get", "new_id", &reply)
```

## 如何进行进程间通信 IPC 呢

首先 linux 下的 FIFO（有名管道）是什么样子的

```bash
mkfifo tpipe
ll
# total 8
# drwx------  2 vagrant vagrant 4096 May 18 17:28 ./
# drwxrwxrwt 10 root    root    4096 May 18 17:28 ../
# prw-rw-r--  1 vagrant vagrant    0 May 18 17:28 tpipe|

# 现在一个terminal
cat tpipe
# 另一个terminal
echo tttttttttt > tpipe
```

此时第一个 terminal 会输出 tttttttttt，另一个命令行会返回

在 Go 中应该如何使用

```go
rpcSvrCmd := exec.Command(conf.RPCSvrBinPath)
rpcSvrStdinPipe, err := rpcSvrCmd.StdinPipe()
rpcSvrStdoutPipe, err := rpcSvrCmd.StdoutPipe()
rpcSvrStderrPipe, err := rpcSvrCmd.StderrPipe()
rpcSvrCmd.Start()
```

如此即可获得子进程的各种 FIFO

# 完整例子

代码见 [exfly/go-ipc](https://github.com/exfly/cslab/tree/master/Code/Go/go-ipc)

```bash
make task rpc && ./bin/task
# ▶ running gofmt…
# ▶ running golint…
# ▶ building executable…
# ▶ building executable…
# 2019/05/19 01:33:12 sleep 1s to wait rpc server startup
# 2019/05/19 01:33:13 2019/05/19 01:33:12 Serving RPC server on {unix rpc.sock}
# 2019/05/19 01:33:13 Finish App:  {Finish App Started}
# 2019/05/19 01:33:13 2019/05/19 01:33:13 stop the server
# 2019/05/19 01:33:13 2019/05/19 01:33:13 deleted socket file: rpc.sock
# 2019/05/19 01:33:13 pipe rpc_srv_stderr has Closed
# 2019/05/19 01:33:13 pipe rpc_srv_stdout has Closed
# 2019/05/19 01:33:13 stop subproc ./bin/task-rpc success
```

当前更新主要以发布代码为主，没有详细解释，详细见代码 [exfly/go-ipc](https://github.com/exfly/cslab/tree/master/Code/Go/go-ipc)