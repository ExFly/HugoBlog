---
title: "Linux0.11源码学习环境配置与相关资源汇总"
author: "Exfly"
cover: "/media/img/default.png"
tags: ["Linux", "资源"]
date: 2016-10-30T14:17:49+08:00
---

记录学习Linux操作系统实现时候使用过的资源，一部分笔记，以及调试中用到的技巧。内容有点乱，仅供个人使用。

<!--more--> 

# linux history

* linux 0.11
* linux 0.95    实现虚拟文件系统
* linux 0.96    实现网络接口

# linux

linux采用分段+分页机制结合管理内存

# linux 调试方法 

* [linux0.11 调试方法](https://wwssllabcd.github.io/blog/2012/08/03/compile-linux011/)
```
gdb tools/system
target remote localhost:1234


gdb常用命令
b: 下中斷點
info b :u 列出目前中断点，也可简写成"i b"
continue(c) 继续执行直到下一个中断点或结束
list(l): 列出目前上下文
step(s): 单步 (会进入 funciton)
next(n) : 单步 (不会进入 funciton)
until(u) 跳离一个 while for 循环
print(p): 显示某变量，如 p str
info register(i r) : 显示 CPU 的 register

GDB 打印出内存中的內容，格式為 x/nyz，其中
n: 要印出的數量
y: 显示的格式，可为C( char), d(整数), x(hex)
z: 单位，可为 b(byte), h(16bit), w(32bit)

cgdb	可显示为上半部分为代码，下半部分命令部分
cgdb tools/system* [linux-0.11启动过程描述](http://labrick.cc/2015/08/13/linux-0-11-boot/)
* [Linux0.11启动过程](http://linux.chinaunix.net/techdoc/install/2007/04/10/954810.shtml)
* [80386保护模式的本质](http://www.jianshu.com/p/1cea7dc5d6b7)
* [linux虚拟地址到线性地址的转化](http://luodw.cc/2016/02/17/address/)
* [Linux内存寻址之分段机制-linux回避了分段机制](http://blog.xiaohansong.com/2015/10/03/Linux%E5%86%85%E5%AD%98%E5%AF%BB%E5%9D%80%E4%B9%8B%E5%88%86%E6%AE%B5%E6%9C%BA%E5%88%B6/)
* [Linux内存寻址之分页机制/](http://blog.xiaohansong.com/2015/10/05/Linux%E5%86%85%E5%AD%98%E5%AF%BB%E5%9D%80%E4%B9%8B%E5%88%86%E9%A1%B5%E6%9C%BA%E5%88%B6/)
* [逻辑地址、线性地址、物理地址和虚拟地址](http://www.voidcn.com/blog/will130/article/p-5705051.html)
* [Intel 80386 程序员参考手册](http://www.kancloud.cn/wizardforcel/intel-80386-ref-manual/123838)
* [linux0.11内核之文件系统](http://harpsword.leanote.com/post/Untitled-563d6103ab6441584f000164)
```

# source

# 80386内存访问公式
32位线性地址 = 段基地址(32位)  + 段内偏移(32位)

48bit = 16 + 32
16位地段选择子 + 32虚拟地址 -> 32线性地址
32线性地址 -> 物理地址
