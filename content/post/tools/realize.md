---
title: "Realize代码分析"
author: "exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["tagA", "tagB"]
date: 2018-10-11T12:55:03+08:00
---

[realize](https://github.com/oxequa/realize) 代码分析

<!--more-->

# Realize

[realize](https://github.com/oxequa/realize) 是 Go 写的 workfloaw 工具，可以配置自己的工作流。项目在修改之后需要进行编译、测试，可以还有其他的一系列流程需要走，可以使用 realize 进行自动化。抽点时间研究了一下源码。这里总结一下思路，不是很完整的解释，简单说一下思路。

## 原理

从 Linux 2.6.13 内核开始，Linux 就推出了 inotify，允许监控程序打开一个独立文件描述符，并针对事件集监控一个或者多个文件，例如打开、关闭、移动/重命名、删除、创建或者改变属性。glib 对对此进行了封装[glib/inotify.h](https://github.com/lattera/glibc/blob/master/sysdeps/unix/sysv/linux/sys/inotify.h),同时各个操作系统都有对应的实现，win 下的 ReadDirectoryChangesW，mac 下的 FSEvents。同时 go 下已经有写好的封装库[fsnotify/fsnotify](https://github.com/fsnotify/fsnotify)，对不同的平台进行了封装。

简单来讲，内核为应用程序提供了系统级文件修改事件的监视器。当文件进行修改后，会通知应用程序监视的文件已经修改了，之后有realize进行事件的处理即可。

比较有意思的是，yaml文件的marshal和unmarshal。之后可以研究一下。

## 核心代码

```golang
// github.com/oxequa/realize/realize/projects.go
func (p *Project) Watch(wg *sync.WaitGroup) {
	var err error
	// change channel
	p.stop = make(chan bool)
	// init a new watcher
	p.watcher, err = NewFileWatcher(p.parent.Settings.Legacy)
	if err != nil {
		log.Fatal(err)
	}
	defer func() {
		close(p.stop)
		p.watcher.Close()
	}()
	// before start checks
	p.Before()
	// start watcher
	go p.Reload("", p.stop)
L:
	for {
		select {
		case event := <-p.watcher.Events():
			if p.parent.Settings.Recovery.Events {
				log.Println("File:", event.Name, "LastFile:", p.last.file, "Time:", time.Now(), "LastTime:", p.last.time)
			}
			if time.Now().Truncate(time.Second).After(p.last.time) {
				// switch event type
				switch event.Op {
				case fsnotify.Chmod:
				case fsnotify.Remove:
					p.watcher.Remove(event.Name)
					if p.Validate(event.Name, false) && ext(event.Name) != "" {
						// stop and restart
						close(p.stop)
						p.stop = make(chan bool)
						p.Change(event)
						go p.Reload("", p.stop)
					}
				default:
					if p.Validate(event.Name, true) {
						fi, err := os.Stat(event.Name)
						if err != nil {
							continue
						}
						if fi.IsDir() {
							filepath.Walk(event.Name, p.walk)
						} else {
							// stop and restart
							close(p.stop)
							p.stop = make(chan bool)
							p.Change(event)
							go p.Reload(event.Name, p.stop)
							p.last.time = time.Now().Truncate(time.Second)
							p.last.file = event.Name
						}
					}
				}
			}
		case err := <-p.watcher.Errors():
			p.Err(err)
		case <-p.exit:
			p.After()
			break L
		}
	}
	wg.Done()
}
```

# Reference

- [用 inotify 监控 Linux 文件系统事件](https://www.ibm.com/developerworks/cn/linux/l-inotify/index.html)
- [oxequa/realize](https://github.com/oxequa/realize)
- [fsnotify/fsnotify](https://github.com/fsnotify/fsnotify)
