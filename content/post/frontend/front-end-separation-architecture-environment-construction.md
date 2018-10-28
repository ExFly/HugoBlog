---
title: "前后端分离架构的Vue环境搭建指南"
author: "Author Name"
cover: "/media/img/frontend/vue-logo.png"
tags: ["前后端分离", "Vue"]
date: 2018-10-28T09:47:12+08:00
---

使用 Vue 前后端+Go 后端，基于 webpack 代理转发，配置前后端分离架构开发环境

<!--more-->

[原文地址](https://github.com/ExFly/HugoBlog/blob/master/content/post/frontend/front-end-separation-architecture-environment-construction.md)

# Web 研发模式演变

最近研究一下前后端的开发模式，看到一个很好的入门路径[developer-roadmap: frontend backend DevOps](https://github.com/kamranahmedse/developer-roadmap),可以看一下，效果还是不错的。

之前看到一个说[web 研发演进](https://github.com/lifesinger/blog/issues/184)这里总结一下。

很久之前，前后端的分工是，前端从设计师那里拿到设计图纸，转化静态页面模板，由后端工程师进行数据库设计等一系列设计之后，套前端给的模板。如上也即后端渲染。

但是这样的流程，所有工作的 Block 在后端，想进一步提高研发速度，应该如何分工？到如今给出的答案是基于 Nodejs 的前后端分离架构。这时前后端分工是这样的：

## 前端的工作

- UI 设计
- 前端路由设计
- 处理浏览器层的展现逻辑

通过 CSS 渲染样式，通过 JavaScript 添加交互功能，HTML 的生成也可以放在这层，具体看应用场景

## 后端的工作

- 业务逻辑和 API 的设计和实现
- 数据库设计和维护
- 后端缓存设计

## 前后端分离下协作体系

前后端分离下的协作方式一般是，前后端各司其职，互不影响。

首先，对于后端来说，后端的主要工作依然是传统的数据库设计、业务逻辑设计，但不需要套模板了，而是为前端提供数据接口

其次，对于前端来说，前端的主要工作是，前端的 ui，以及获取数据，在前端渲染。

工作流程是，先进行 API 设计。前后端一起设计数据接口以及数据返回的格式，现在比较常见的是 json 数据。可以根据接口生成一些 mock 用的 json 数据文件，供前端开发使用。后端根据这个 API 规范实现真正的接口。两端分别并行开发。开发结束时候联调，打通前后端之后进行调试。

具体，可以看一下[网易前后端分离实践](https://github.com/genify/ita1024/blob/master/%E7%BD%91%E6%98%93%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E5%AE%9E%E8%B7%B5.md).

# 基于 Vue 前后端分离环境搭建

这里对前后端分离 Vue 的开发环境进行演示。思路是，前后端分离，后端可以设置 cookie，前端可以接收到配置的 cookie

### node 环境安装

安装方法见[这里](http://www.runoob.com/nodejs/nodejs-install-setup.html)

### Vue 安装

```sh
npm config set registry 'https://registry.npm.taobao.org'
npm install -g @vue/cli
vue init webpack demo
cd demo
npm dev run
```

即可在浏览器中看到效果，熟悉的 vue 页面

### 安装 axios，实现前后端交互，并实现后端设置 cookie，在前端可以生效

安装 axios

```sh
npm install axios
```

修改/src/components/HelloWorld.vue 中对应的 srcipt

```js
<script>
import axios from 'axios'

axios.get('/sc') // 使用ajax
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  }
}
</script>
```

最终完成的时候，访问 '/'，可以看到 cookie 添加了一个 kv 对。现在暂时看不到效果，因为接口后端没有实现。

后端接口实现,这里使用的 go，具体 go 编译器的安装方法[见这里](https://golang.org/doc/install#install)

```go
package main

import (
	"log"
	"net/http"
	"time"
)

func LoggingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		log.Println("->", r.URL)
		next.ServeHTTP(w, r)
		log.Println("<-")
	})
}

func main() {
	http.Handle("/sc", LoggingMiddleware(http.HandlerFunc(indexHandler)))
	port := ":8081"
	log.Println("starting on http://localhost" + port)
	log.Fatal(http.ListenAndServe(port, nil))
}

func indexHandler(w http.ResponseWriter, req *http.Request) {
	expire := time.Now().AddDate(0, 0, 1)
	cookie := http.Cookie{Name: "csrftoken", Value: expire.String(), Expires: expire}
	http.SetCookie(w, &cookie)
}
```

此时前后端都已经实现了，但是因为前端开在了端口 8080，后端开在了 8081，涉及到跨域，相互没法访问。需要配置一下 webpack 的配置才可以。

```js
// 修改一下文件/config/index.js: 13 line
proxyTable: {
    '/': {
    target: 'http://localhost:8081',
    changeOrigin: true
    }
},
```

之后，开启前后端服务：

```sh
npm dev run
go run server.go
```

之后访问前端页面，localhost:8080，按 F12 -> Application -> Cookies， 即可看到每次刷新都会改变的 csrftoken 的 cookie

![前后度分离结果，设置了cookie](/media/img/frontend/setcookie-front-back-sep.png)

# 后记

既然在前后端分离后的后端可以配置cookie，其他的所有操作都可以进行了。进一步可以实现其他的操作。如上。

## 引用
- [developer-roadmap: frontend backend DevOps](https://github.com/kamranahmedse/developer-roadmap)
- [web 研发演进](https://github.com/lifesinger/blog/issues/184)
- [网易前后端分离实践](https://github.com/genify/ita1024/blob/master/%E7%BD%91%E6%98%93%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E5%AE%9E%E8%B7%B5.md)
