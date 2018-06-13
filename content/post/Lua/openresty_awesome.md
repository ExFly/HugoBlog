---
title: "OpenResty最佳实践"
author: "Exfly"
cover: "/media/img/Lua/openresty_awesome/icon.jpg"
tags: ["Lua", "OpenResty"]
date: 2018-06-12T16:01:38+08:00
---

如果你正在学Lua与openresty，那你就一定知道在开发过程中，调试代码、单元测试是多么的麻烦。这里整理了一些lua开发的最佳实践。

<!--more--> 

# 简介
openresty中lua ide调试，单元测试比较麻烦；lua对库的管理比较散漫。在公司生产环境，一般没有外网环境，OpenResty的安装和lua项目的部署都比较麻烦。
结合Python的一些经验，在这里整理一下自己对Lua的理解，以及Lua最佳实践。

# OpenResty安装
对于软件，使用编译方式安装比较好，比如Ubuntu，apt-get安装的包一般都会比较旧。如下介绍我的编译参数。这里需要自己下载自己的依赖包：naxsi, nginx-goodies-nginx-sticky-module-ng，pcre，openssl，zlib，并根据我的配置进行修改相应参数
```sh
./configure --prefix=$HOME/openresty \
 --add-module=$HOME/openresty/setupfile/third/naxsi-0.55.3/naxsi_src \
 --add-module=$HOME/openresty/setupfile/third/nginx-goodies-nginx-sticky-module-ng \
 --with-pcre=$HOME/openresty/setupfile/depency/pcre-8.41 \
 --with-openssl=$HOME/openresty/setupfile/depency/openssl-1.0.2k \
 --with-zlib=$HOME/openresty/setupfile/depency/zlib-1.2.11 \
 --with-http_v2_module \
 --with-http_sub_module \
 --with-http_stub_status_module \
 --with-http_realip_module \
 --with-cc-opt=-O2 \
 --with-luajit
```

* [这里](http://blog.51cto.com/1992tao/1865611)是一个比较好的 nginx的笔记，可以过一遍
* 我在学习的时候，看了这本书[深入理解Nginx模块开发与架构解析](#),毕竟讲的比较系统，可以借鉴一下
* 有问题，知乎，搜索引擎

## 安装luarocks
* 下载地址 http://luarocks.github.io/luarocks/releases/
* 编译安装
```
./configure --prefix=$HOME/openresty/luajit \
    --with-lua=$HOME/openresty/luajit \
    --lua-suffix=jit \
    --with-lua-include=$HOME/openresty/luajit/include/luajit-2.1

--prefix 设定 luarocks 的安装目录
--with-lua 则是系统中安装的 lua 的根目录
--lua-suffix 版本后缀，此处因为openresyt的lua解释器使用的是 luajit ,所以此处得写 jit
--with-lua-include 设置 lua 引入一些头文件头文件的目录
make build && make install
```

# lua面向对象
lua 借助table以及metatable的概念进行oo的。这里摘了一个博客的代码，看起来还可以。以后可以使用这个。[Lua 中实现面向对象](https://blog.codingnow.com/2006/06/oo_lua.html)。
这里要说一下lua中[.运算和:的区别](https://www.kancloud.cn/digest/luanote/119940)，a={};a.fun(a, arg) 等价于 a:fun(arg)，其实就是`:`可以省略self参数。
```lua
local _class={}
function class(super)
    local class_type={}
    class_type.ctor=false
    class_type.super=super
    class_type.new=function(...) 
            local obj={}
            do
                local create
                create = function(c,...)
                    if c.super then
                        create(c.super,...)
                    end
                    if c.ctor then
                        c.ctor(obj,...)
                    end
                end
                create(class_type,...)
            end
            setmetatable(obj,{ __index=_class[class_type] })
            return obj
        end
    local vtbl={}
    _class[class_type]=vtbl
 
    setmetatable(class_type,{__newindex=
        function(t,k,v)
            vtbl[k]=v
        end
    })
    if super then
        setmetatable(vtbl,{__index=
            function(t,k)
                local ret=_class[super][k]
                vtbl[k]=ret
                return ret
            end
        })
    end
 
    return class_type
end
```

# 基本编码规范 设计
[可以参考OpenResty的最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/web/code_style.html)，平时用起来，大部分跟c的风格差不多吧。主要是所使用的代码风格要统一。

# 包管理
lua下有两个包管理系统，LuaDist和LuaRocks

# 单元测试
* **重点** 如下方法请在命令行中使用类似`curl localhost/unittest`进行测试，浏览器中看会很痛苦
* [OpenResty最佳实践-单元测试](https://moonbingbing.gitbooks.io/openresty-best-practices/test/unittest.html)给出一种方法。我的处理方法是，在nginx.conf中的server中建一个单独的location，content_by_lua_file 设置unittest.lua。公司用的verynginx，所以我把此配置放到了router.lua中(当然配置方法类似，这个很容易研究，就不放到这里了)。

```lua
-- file: unittest.lua
local _M = {}
local csrf_test = require("test.test_csrf")
local tmp_test = require("test.tmp_test")
function _M:run_unittest()
    csrf_test:run()
end
return _M

-- file: test_csrf.lua
local iresty_test = require("resty.iresty_test")
local json = require("json")
local config = require("config")
local csrf_config = require("csrf_config")
local token = require("token")
local tabletls = require("tabletls")
local tb = iresty_test.new({unit_name="test_csrf"})

local function assert_eq(wanted, real, msg)
    if wanted ~= real then
        error(msg or "error", 2) -- 请注意参数 2
    end
end
local function assert_not_eq(wanted, real, msg)
    if wanted == real then
        error(msg or "error", 2)
    end
end
function tb:test_geturl()
	assert_eq("/unittest", ngx.var.uri, "the unittest url changed")
end
function tb:run_unittest()
    tb:run()
end
return tb
```
* 如上有一个很有意思的地方，`error(msg or "error", 2)`,其中的2有些讲究，表示返回调用函数所在行，还有0（忽略行号），1（error调用位置行号）
* 性能测试 代码覆盖率 API测试等，都可以去[OpenResty最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/)中找，配置很简单。

# 远程调试 OpenResty
* 对于此部分，对于有些人来说，使用日志就已经足够了。可对于有些时候，在代码中太多的日志有不利于维护。这里自己要尽力多好日志和调试的平衡吧。
* 此调试方法适用于 win linux osx
* 先贴这里用到的luaIDE地址：[ZeroBraneStudio](https://github.com/pkulchenko/ZeroBraneStudio)
* 如下为安装步骤：
* 下载这个项目，[ZeroBraneStudio](https://github.com/pkulchenko/ZeroBraneStudio/archive/master.zip)，解压可以直接用【调试方法在下载好的文件中README.md中有相应的链接】
* 启动ZBS，Project -> Start Debugger Server
* 复制<ZBS>/lualibs/mobdebug/mobdebug.lua -> nginx lua path,
* 复制<ZBS>/lualibs/socket.lua -> nginx lua path，
* 复制<ZBS>/bin/clibs/socket/core -> socket设为nginx lua cpath（调试时候，使用的是require("socket.core")形式导入包。这里需要注意core文件后缀，win是dll，linux是so，）
* nginx配置好,将如上依赖加到nginx.conf中，让lua可以找到这些文件即可
* 创建需要调试的lua文件
```
require('mobdebug').start('192.168.1.22')
local name = ngx.var.arg_name or "Anonymous"
ngx.say("Hello, ", name, "!")
ngx.say("Done debugging.")
require('mobdebug').done()
```
注：start()呼叫需要运行IDE的计算机的IP 。默认情况下使用“localhost”，但是由于您的nginx实例正在运行，因此您需要指定运行IDE的计算机的IP地址（在我的例子中192.168.1.22）

* 在ide中打开需要调试的如上lua文件
* Project -> Project Directory -> Set From Current File。
* 此时，打开浏览器，访问需要此文件处理的链接
* 此时开始调试
![ZeroBrane Studio调试OpenResty](/media/img/Lua/openresty_awesome/debug.png)
* 注：在最下侧有Remote console，在这里可以执行任何ngx lua语句
* 如上流程没有截图，或者没有说清楚，可以来[这里](http://ju.outofmemory.cn/entry/326200)

# nginx一些技巧
* 看我配置的nginx.conf
```
lua_package_path '$prefix/lua_script/?.lua;;';
```
* [我的笔记](https://github.com/ExFly/CsLearning/blob/master/NoteBookForDevelop/%E6%96%87%E6%A1%A3/Server/nginx_lua.md)

# 资料
* [章亦春](http://agentzh.org/)
* [OpenResty](https://openresty.org/)
* [OpenResty最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices)
* [Lua 5.1 参考手册](https://www.codingnow.com/2000/download/lua_manual.html)
* [Lua 5.3 参考手册](http://cloudwu.github.io/lua53doc/contents.html)
* [云风 github](https://github.com/cloudwu)
* [awesome-lua](https://github.com/LewisJEllis/awesome-lua)
* [awesome-resty](https://github.com/bungle/awesome-resty)
* [Nginx-Lua-OpenResty-Resources](https://github.com/fcambus/nginx-resources)A collection of resources covering Nginx, Nginx + Lua, OpenResty and Tengine
