---
title: "Go Overview"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2020-05-06T08:57:31+08:00
draft: true
---

文章简介：golang 各种元素整理

<!--more-->

## 编译原理

> 比较好的介绍 golang 的编译器看 [这里](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/)

代码中对 `go compiler` 的描述: [src/cmd/compile/README.md](https://github.com/golang/go/blob/master/src/cmd/compile/README.md)

what is ssa: [wiki](https://en.wikipedia.org/wiki/Static_single_assignment_form) IR

golang Static Single Assignment: [src/cmd/compile/internal/ssa/README.md](https://github.com/golang/go/blob/master/src/cmd/compile/internal/ssa/README.md)

### 编译器入口 [src/cmd/compile/internal/gc/main.go](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/main.go)

### 技巧

- 生成 SSA 整个过程结果展示(html): GOSSAFUNC=hello go build hello/hello.go
- 生成 plan9 汇编代码 GOOS=linux GOARCH=amd64 go tool compile -S hello/hello.go

TODO: 给一个例子，并解释一下(需要了解所有步骤)

## plan9

## 基本数据结构

### 数组

https://github.com/golang/go/blob/f07059d949057f414dd0f8303f93ca727d716c62/src/cmd/compile/internal/gc/sinit.go#L875-L967

当元素数量小于或者等于 4 个时，会直接将数组中的元素放置在栈上；
当元素数量大于 4 个时，会将数组中的元素放置到静态区并在运行时取出；

无论是在栈上还是静态存储区，数组在内存中其实就是一连串的内存空间，表示数组的方法就是一个指向数组开头的指针、数组中元素的数量以及数组中元素类型占的空间大小

Go 语言中对数组越界的判断是可以在编译期间由静态类型检查完成的 https://github.com/golang/go/blob/b7d097a4cf6b8a9125e4770b54d33826fa803023/src/cmd/compile/internal/gc/typecheck.go#L327-L2081

### 切片

> [图解 slice](https://blog.golang.org/slices-intro)

[NewSlice](https://github.com/golang/go/blob/616c39f6a636166447bdaac4f0871a5ca52bae8c/src/cmd/compile/internal/types/type.go#L484-L496)

```
// [slice](https://github.com/golang/go/blob/4c003f6b780b471afbf032438eb6c7519458855b/src/reflect/value.go#L1973)
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```

```
slice := []int{1, 2, 3}
slice := make([]int, 10)
```

[:], a slice referencing the storage of x

> [:] 操作是创建切片最底层的一种方法

#### make

[make 关键词](https://github.com/golang/go/blob/b7d097a4cf6b8a9125e4770b54d33826fa803023/src/cmd/compile/internal/gc/typecheck.go#L327-L2126)

当切片发生逃逸或者非常大时，我们需要 [runtime.makeslice](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/runtime/slice.go#L34-L50) 函数在堆上初始化，
如果当前的切片不会发生逃逸并且切片非常小的时候，make([]int, 3, 4) 会被直接转换成如下所示的代码：

```
var arr [4]int
n := arr[:3]
```

[golang 对 slice 比较有意义的优化](https://github.com/golang/go/commit/020a18c545bf49ffc087ca93cd238195d8dcc411)

#### append

[runtime.growslice](https://github.com/golang/go/blob/440f7d64048cd94cba669e16fe92137ce6b84073/src/runtime/slice.go#L76-L191)

1. 如果期望容量大于当前容量的两倍就会使用期望容量；
2. 如果当前切片容量小于 1024 就会将容量翻倍；
3. 如果当前切片容量大于 1024 就会每次增加 25% 的容量，直到新容量大于期望容量；

### 哈希表

#### 哈希函数

#### 冲突解决方法

开放寻址法和拉链法

```
[hmap](https://github.com/golang/go/blob/ed15e82413c7b16e21a493f5a647f68b46e965ee/src/runtime/map.go#L115-L129)
type hmap struct {
	count     int
	flags     uint8
	B         uint8
	noverflow uint16
	hash0     uint32

	buckets    unsafe.Pointer
	oldbuckets unsafe.Pointer
	nevacuate  uintptr

	extra *mapextra
}
```

**创建 map**

[runtime.makemap](https://github.com/golang/go/blob/dcd3b2c173b77d93be1c391e3b5f932e0779fb1f/src/runtime/map.go#L303-L336)

golang 通过拉链法处理冲突

**读写**
增加、删除和修改
[mapaccess1](https://github.com/golang/go/blob/36f30ba289e31df033d100b2adb4eaf557f05a34/src/runtime/map.go#L394-L450)

扩容（不是原子操作）

[runtime.mapdelete](https://github.com/golang/go/blob/36f30ba289e31df033d100b2adb4eaf557f05a34/src/runtime/map.go#L685-L791)

Go 语言使用拉链法来解决哈希碰撞的问题实现了哈希表，它的访问、写入和删除等操作都在编译期间转换成了运行时的函数或者方法。

哈希在每一个桶中存储键对应哈希的前 8 位，当对哈希进行操作时，这些 tophash 就成为了一级缓存帮助哈希快速遍历桶中元素，每一个桶都只能存储 8 个键值对，一旦当前哈希的某个桶超出 8 个，新的键值对就会被存储到哈希的溢出桶中。

随着键值对数量的增加，溢出桶的数量和哈希的装载因子也会逐渐升高，超过一定范围就会触发扩容，扩容会将桶的数量翻倍，元素再分配的过程也是在调用写操作时增量进行的，不会造成性能的瞬时巨大抖动。

### 字符串

[十分钟搞清字符集和字符编码](http://cenalulu.github.io/linux/character-encoding/)

```go
type StringHeader struct {
    Data uintptr
    Len int
}

type stringStruct struct {
	str unsafe.Pointer
	len int
}
```

[slicebytetostring](https://github.com/golang/go/blob/b6feb03b24a75164438c3419c0bc01fef62825a0/src/runtime/string.go#L80)

## OO

### 函数调用

C 语言和 Go 语言在设计函数的调用惯例时选择也不同的实现。C 语言同时使用寄存器和栈传递参数，使用 eax 寄存器传递返回值；而 Go 语言使用栈传递参数和返回值。我们可以对比一下这两种设计的优点和缺点：

1. C 语言的方式能够减少大量小函数调用的开销，但是也增加了实现的复杂度；
   - CPU 访问栈(内存)的开销比[访问寄存器高几十倍](https://arstechnica.com/gadgets/2002/07/caching/2/)；
   - 需要单独处理函数参数过多的情况；
2. Go 语言的方式能够降低实现的复杂度并支持多返回值，但是牺牲了函数调用的性能；
   - 不需要考虑超过寄存器数量的参数应该如何传递；
   - 不需要考虑不同架构上的寄存器差异；
   - 函数入参和出参的内存空间需要在栈上进行分配；

Go 语言使用栈作为参数和返回值传递的方法是综合考虑后的设计，选择这种设计意味着编译器会更加简单、更容易维护。

#### 参数传递

golang 无论是传递基本类型、结构体还是指针，都会对传递的参数进行拷贝

### 接口

类型转换、类型断言以及动态派发机制

iface 结构体 是带有一组方法的接口
eface 结构体 不带任何方法的 interface{}

```
type eface struct { // 16 bytes
	_type *_type
	data  unsafe.Pointer
}

type iface struct { // 16 bytes
	tab  *itab
	data unsafe.Pointer
}
```

动态派发的过程只是放大了参数拷贝带来的影响, 用结构体实现接口会有更多消耗(125%)

### 反射

## 常用关键字

### for 和 range

一些 golang buildin func 是由原始的汇编写成，比如[runtime·memclrNoHeapPointers](https://github.com/golang/go/blob/05c02444eb2d8b8d3ecd949c4308d8e2323ae087/src/runtime/memclr_386.s#L12-L16)

### select

当 select 中的两个 case 同时被触发时，就会随机选择一个 case 执行。

1. select 能在 Channel 上进行非阻塞的收发操作；
2. select 在遇到多个 Channel 同时响应时会随机挑选 case 执行；(如果我们按照顺序依次判断，那么后面的条件永远都会得不到执行，而随机的引入就是为了避免饥饿问题的发生)

非阻塞的收发:

```
select {
case err := <-errCh:
    return err
default:
    return nil
}
```

[scase](https://github.com/golang/go/blob/d1969015b4ac29be4f518b94817d3f525380639d/src/runtime/select.go#L28-L34)

```
type scase struct {
	c           *hchan         // chan
	elem        unsafe.Pointer // data element
	kind        uint16
	pc          uintptr // race pc (for race detector / msan)
	releasetime int64
}
```

### defer

### panic 和 recover

[Defer, Panic, and Recover][defer-panic-recover]

1. panic 只会触发当前 Goroutine 的延迟函数调用；
2. recover 只有在 defer 函数中调用才会生效；
3. panic 允许在 defer 中嵌套多次调用；

4. 跨协程失效 首先要展示的例子就是 panic 只会触发当前 Goroutine 的延迟函数调用

多个 Goroutine 之间没有太多的关联
[runtime.\_panic](https://github.com/golang/go/blob/cfe3cd903f018dec3cb5997d53b1744df4e53909/src/runtime/runtime2.go#L891-L900)

### 总结

1. 编译器会负责做转换关键字的工作；
   - 将 panic 和 recover 分别转换成 runtime.gopanic 和 runtime.gorecover；
   - 将 defer 转换成 deferproc 函数；
   - 在调用 defer 的函数末尾调用 deferreturn 函数；
2. 在运行过程中遇到 gopanic 方法时，会从 Goroutine 的链表依次取出 \_defer 结构体并执行；
3. 如果调用延迟执行函数时遇到了 gorecover 就会将 \_panic.recovered 标记成 true 并返回 panic 的参数；
   - 在这次调用结束之后，gopanic 会从 \_defer 结构体中取出程序计数器 pc 和栈指针 sp 并调用 recovery 函数进行恢复程序；
   - recovery 会根据传入的 pc 和 sp 跳转回 deferproc；
   - 编译器自动生成的代码会发现 deferproc 的返回值不为 0，这时会跳回 deferreturn 并恢复到正常的执行流程；
4. 如果没有遇到 gorecover 就会依次遍历所有的 \_defer 结构，并在最后调用 fatalpanic 中止程序、打印 panic 的参数并返回错误码 2；

### make 和 new

make 关键字的作用是创建切片、哈希表和 Channel 等内置的数据结构，而 new 的作用是为类型申请一片内存空间，并返回指向这片内存的指针。

## 运行时

### 并发

#### 上下文 Context

1. Deadline — 返回 context.Context 被取消的时间，也就是完成工作的截止日期；
2. Done — 返回一个 Channel，这个 Channel 会在当前工作完成或者上下文被取消之后关闭，多次调用 Done 方法会返回同一个 Channel；
3. Err — 返回 context.Context 结束的原因，它只会在 Done 返回的 Channel 被关闭时才会返回非空的值；
   1. 如果 context.Context 被取消，会返回 Canceled 错误；
   2. 如果 context.Context 超时，会返回 DeadlineExceeded 错误；
4. Value — 从 context.Context 中获取键对应的值，对于同一个上下文来说，多次调用 Value 并传入相同的 Key 会返回相同的结果，该方法可以用来传递请求特定的数据；

[Go Concurrency Patterns: Context](https://blog.golang.org/context)

#### 同步原语与锁

Mutex、RWMutes、WaitGroup、Once x/sync/errgroup.Group、x/sync/semaphore.Weighted、x/sync/singleflight.Group 和 x/sync/syncmap.Map

[sync.Mutex](https://github.com/golang/go/blob/71239b4f491698397149868c88d2c851de2cd49b/src/sync/mutex.go#L25-L28)

```
type Mutex struct {
	state int32
	sema  uint32
}
```

正常模式 => 在正常模式下，锁的等待者会按照先进先出的顺序获取锁。但是刚被唤起的 Goroutine 与新创建的 Goroutine 竞争时，大概率会获取不到锁，为了减少这种情况的出现，一旦 Goroutine 超过 1ms 没有获取到锁，它就会将当前互斥锁切换饥饿模式，防止部分 Goroutine 被『饿死』。
饥饿模式 => 在饥饿模式中，互斥锁会直接交给等待队列最前面的 Goroutine。新的 Goroutine 在该状态下不能获取锁、也不会进入自旋状态，它们只会在队列的末尾等待。如果一个 Goroutine 获得了互斥锁并且它在队列的末尾或者它等待的时间少于 1ms，那么当前的互斥锁就会被切换回正常模式。

相比于饥饿模式，正常模式下的互斥锁能够提供更好地性能，饥饿模式的能避免 Goroutine 由于陷入等待无法获取锁而造成的高尾延时。

Go 语言还在子仓库 sync 中提供了四种扩展原语，x/sync/errgroup.Group、x/sync/semaphore.Weighted、x/sync/singleflight.Group 和 x/sync/syncmap.Map，其中的 x/sync/syncmap.Map 在 1.9 版本中被移植到了标准库中。

#### 定时器

而在 10ms 的这个粒度下，作者在社区中也没有找到能够使用的计时器实现，一些使用时间轮算法的开源库也不能很好地完成这个任务。

#### Channel

创建、发送、接收和关闭
不要通过共享内存的方式进行通信，而是应该通过通信的方式共享内存
CSP
无锁（lock-free）队列更准确的描述是使用乐观并发控制的队列
[runtime.hchan](https://github.com/golang/go/blob/e35876ec6591768edace6c6f3b12646899fd1b11/src/runtime/chan.go#L32)

[impl lock free 论文](http://people.cs.pitt.edu/~jacklange/teaching/cs2510-f12/papers/implementing_lock_free.pdf)
[社区 lock free chan](https://github.com/OneOfOne/lfchan)

**send**
[runtime.chansend1](https://github.com/golang/go/blob/e35876ec6591768edace6c6f3b12646899fd1b11/src/runtime/chan.go#L126)

1. 当存在等待的接收者时，通过 runtime.send 直接将数据发送给阻塞的接收者；
2. 当缓冲区存在空余空间时，将发送的数据写入 Channel 的缓冲区；
3. 当不存在缓冲区或者缓冲区已满时，等待其他 Goroutine 从 Channel 接收数据；

**recv**

1. 如果 Channel 为空，那么就会直接调用 runtime.gopark 挂起当前 Goroutine；
2. 如果 Channel 已经关闭并且缓冲区没有任何数据，runtime.chanrecv 函数会直接返回；
3. 如果 Channel 的 sendq 队列中存在挂起的 Goroutine，就会将 recvx 索引所在的数据拷贝到接收变量所在的内存空间上并将 sendq 队列中 Goroutine 的数据拷贝到缓冲区；
4. 如果 Channel 的缓冲区中包含数据就会直接读取 recvx 索引对应的数据；
5. 在默认情况下会挂起当前的 Goroutine，将 runtime.sudog 结构加入 recvq 队列并陷入休眠等待调度器的唤醒；

#### 调度器

每一次线程上下文的切换都需要消耗 ~1us 左右的时间：[Measuring context switching and memory overheads for Linux threads](https://eli.thegreenplace.net/2018/measuring-context-switching-and-memory-overheads-for-linux-threads/)
Go 调度器对 Goroutine 的上下文切换约为 ~0.2us，减少了 80% 的额外开销

#### 网络轮询器

#### 系统监控

### 内存管理

#### 内存分配器
TCMalloc
隔离适应
#### 垃圾收集器

标记清除

三色抽象
1. 从灰色对象的集合中选择一个灰色对象并将其标记成黑色；
2. 将黑色对象指向的所有对象都标记成灰色，保证该对象和被该对象引用的对象都不会被回收；
3. 重复上述两个步骤直到对象图中不存在灰色对象；

屏障技术

#### 栈内存管理

## 其他

### unsafe

### 逃逸分析

## build comment

```
//go:nosplit
//go:linkname
//go:noescape
//go:notinheap
```

```
*(*int)(nil) = 0 // not reached
```

## reference

[defer-panic-recover]: https://blog.golang.org/defer-panic-and-recover "Defer, Panic, and Recover"
