---
title: "Openapi"
author: "Author Name"
cover: "/media/img/Java/openapi/Swagger-logo.png"
tags: ["openapi", "codegen"]
date: 2019-02-04T13:41:19+08:00
---

swagger and openapi 代码生成和文档自动生成一些体会

<!--more-->

# 说在前面

平时使用[gqlgen](https://gqlgen.com)，一般的 workflow 是先写 graphql 的 schema，然后 code generate 对应的 model 和 api 的实现的空接口，自己对应实现对应的 resolver 即可。使用起来很流程。最近调研一下 java 下类似的工具。找到在 java 下 star 最多的项目[graphql-java](https://github.com/graphql-java/graphql-java)，但并不是这里讨论的。
这里讨论的是基于 spring 的 openapi 的实现和 code generate 方案。

# 基于 openapi 的 code generate 方案

首先简单介绍一下 openapi，他是语言无关的 restful 描述语言，可以使用 yaml 进行编写接口文档，通过 generate，生成不同语言的 client 和 server。[这里](https://swagger.io/docs/specification/about/)有官方的简介。自己比较关注的语言是 python、java、go、rust，js。

踩到的一些坑，这里简单试用了一下generate java。首先，maven下有对应的openapi的plugin，[openapi-generator-maven-plugin](https://github.com/OpenAPITools/openapi-generator/blob/master/modules/openapi-generator-maven-plugin/README.md), 但每次compile都会进行generate，并不是我所希望的，所以这里使用了基于docker的使用方案。代码见[exfly:gorgestar/isn](https://github.com/gorgestar/isn/tree/v0.0.1),使用[generator.sh](https://github.com/gorgestar/isn/blob/v0.0.1/generator.sh)生成对应的代码，配置文件可以看[generator.json](https://github.com/gorgestar/isn/blob/v0.0.1/generator.json)，

我的使用策略是，先修改[openapi.yaml](https://github.com/gorgestar/isn/blob/v0.0.1/src/main/resources/openapi.yaml)，创建或者修改restful api，然后 `bash ./generator.sh`，生成对应接口默认空实现，之后由我们对应的实现接口即可。

一些自己没有进行操作的方案：

* 如何保证api和接口文档一致：可以对应的使用swagger的api-doc json进行转换为yaml，对原本手写的`openapi.yaml`进行比较，确认文档的一致性
* 这个版本的generate每次都会将所有的文件覆盖，需要编辑[.openapi-generator-ignore](https://github.com/gorgestar/isn/blob/v0.0.1/.openapi-generator-ignore)，类似gitignore的东西，类比修改即可。被忽略的文件，即使被删除，也不会自动生成对应文件，生成逻辑看起来比较傻。


# 其他资源
* [exfly:gorgestar/isn](https://github.com/gorgestar/isn),本文项目代码
* [swagger](https://swagger.io/) 
* [raml](https://raml.org/)
* [OpenAPI-Specification](https://github.com/OAI/OpenAPI-Specification)
* [openapi-generator](https://github.com/OpenAPITools/openapi-generator)
* [openapi generator maven plugin](https://github.com/OpenAPITools/openapi-generator/blob/master/modules/openapi-generator-maven-plugin/README.md)

* [spring rest docs](https://spring.io/projects/spring-restdocs)
