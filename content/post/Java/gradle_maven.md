---
title: "Gradle和Maven使用方法总结"
author: "Exfly"
cover: "/media/img/Java/gradle_maven/gradle.png"
tags: ["Java"]
date: 2018-06-24T17:38:17+08:00
---

文章简介： <br>1.总结gradle和maven正确使用方法 <br>2.开箱即用maven&gradle同时支持的项目配置。<br>Gradle和Maven使用起来都比较方便，而Gradle使用更灵活，配置更方便。而公司环境一般使用Maven。因此就有了取舍，是迁移到Gradle，还是继续使用Maven？其实不需要纠结，谁说必须取舍的，两个都用起来就是了！！！

<!--more--> 

# 说在前面
Gradle和Maven都是项目自动构建工具，编译源代码只是整个过程的一个方面，更重要的是，你要把你的软件发布到生产环境中来产生商业价值，所以，你要运行测试，构建分布、分析代码质量、甚至为不同目标环境提供不同版本，然后部署。整个过程进行自动化操作是很有必要的。

整个过程可以分成以下几个步骤：

* 编译源代码
* 运行单元测试和集成测试
* 执行静态代码分析、生成分析报告
* 创建发布版本
* 部署到目标环境
* 部署传递过程
* 执行冒烟测试和自动功能测试

两者都是项目工具，但是maven使用的最多，Gradle是后起之秀，想spring等都是使用gradle构建的。Gradle抛弃了Maven的基于XML的繁琐配置，采用了领域特定语言Groovy的配置，大大简化了构建代码的行数。

比如maven要 这么写
```
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-core</artifactId>
	<version>${spring.version}</version>
</dependency>
```
gradle这么写
```
compile('org.springframework:spring-core:2.5.6')
```
详细的Gradle和Maven比较看[这里](https://www.tianmaying.com/tutorial/Gradle)讲的很好了。[gradle官方](https://gradle.org/maven-vs-gradle/)也对两个工具进行了比较。

我们可以使用其中一个，或者两个一起使用！！！这是可行的，当然前提是，有一个人在整个过程中维护相同功能的两份配置。实际上并不难。抽一个周末空余时间，自己把这两个都熟悉了一下，整理了一套Gradle&Maven日常开发中常用的包和插件的集合，作为项目的开始。**比较通用，所以需要根据公司或个人项目实际情况*加入私服*的配置，以及你想使用的*jar包***，如此简单。如果使用过程中遇到什么问题，请联系我。别忘了，帮我star一下。

接下来涉及到的内容：

* maven 正确使用方法
* gradle正确使用方法
* gradle项目和maven项目相互转化
* 一个项目同时支持maven和gradle配置：一个好的开始

# maven 正确使用方法
## maven版本不相同问题
我们大部分时候使用IDE进行项目开发的时候，大部分时候会直接使用IDE创建MAVEN项目，这是正确的。可是，您有没有发现，大家合作的时候，由于maven版本不相同，哪怕是3.5.1和3.5.2的区别，都会引发一场血案！我的可以正常打开项目，而其他人却会出现问题。除了IDE下载包损坏外，就是maven的版本不相同。其实通过一些工具，已经可以让这种情况不在发生，那就是[Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)。请看如下图(图没配错，maven的wrapper和gradle的wrapper流程上完全相同)

![wrapper-workflow](/media/img/Java/gradle_maven/wrapper-workflow.png)

**前提条件**：

* 项目创建者系统中已经由maven的命令
* 其他人没有要求，mvn可有可无（原因之后说）

具体如何做：

* 项目创建者执行 `mvn -N io.takari:maven:wrapper -Dmaven=3.5.3`
	* 此时，项目目录会生成`mvnw.cmd`和`mvnw`，之后的所有操作都是基于此，也就是说，项目开发者不需要由任何依赖，除了jdk-\_!!!
* 项目创建者执行 `mvnw archetype:generate`
	* 此步是自动生成项目目录结构。同时，项目管理者需要搭建好基础的代码框架。之后可以开发了
* 项目开发者 
	* `mvnw.cmd compiler:compile`
	* `mvnw.cmd exec:java -Dexec.mainClass="org.exfly.LombokL.LombokLApplication" -q`
	* `mvn.cmd clean`
	* `mvn.cmd test`
	* 。。。

**注意**:

1. 当第一次执行`mvnw.cmd`时候，会自动下载对应版本的Maven，maven的`$HOME/.m2/wrapper/dists/<version>/`下。
2. 初网络问题，如果出现错误，依赖包已经下好，只需要到`1`所说的位置去掉后缀.pack，重新运行即可。

## 使用[dependencyManagement](https://zhuanlan.zhihu.com/p/31020263)集中管理版本依赖
* [dependencyManagement](https://zhuanlan.zhihu.com/p/31020263)这里已经很好的解释如何做。同时可以借鉴springboot-parent

## 多模块项目管理方法
* [多模块项目的POM重构](http://www.infoq.com/cn/news/2011/01/xxb-maven-3-pom-refactoring)
* 通过parent的方式，将多模块依赖集中管理，

## 如何更好的使用maven进行项目管理 几点建议
* 尽量使用wapper多
* 使用[dependencyManagement](https://zhuanlan.zhihu.com/p/31020263)集中管理版本依赖
* bin下有mvn和mvnDebug(运行mvn时开始debug)
* M2_HOME maven主程序的安装目录
* ~/.m2 本地包下载位置
* http代理
	* setting.xml中的proxies
* MAVEN_OPTS
	* 运行mvn时候相当于运行java命令，MAVEN_OPTS可以配置为任何java的命令参数
* 设置MAVEN_OPTS环境变量
* 配置用户范围settings.xml
	* %M2_HOME%/conf/settings.xml 为全局配置文件
	* ~/.m2/settings.xml 为用户配置文件
* 不要使用IDE内嵌的Maven，应该配置IDE中为自己安装的maven
* 显示声明所有用到的依赖

## [我的maven常用命令笔记](https://github.com/ExFly/CsLearning/blob/master/NoteBookForDevelop/%E6%96%87%E6%A1%A3/Java/%E5%B7%A5%E5%85%B7/Maven.md)
[我的maven常用命令笔记](https://github.com/ExFly/CsLearning/blob/master/NoteBookForDevelop/%E6%96%87%E6%A1%A3/Java/%E5%B7%A5%E5%85%B7/Maven.md)

# gradle正确使用方法
理由同上节，直接说使用方法。可以对照[我的笔记查看](https://github.com/ExFly/CsLearning/blob/master/NoteBookForDevelop/%E6%96%87%E6%A1%A3/Java/%E5%B7%A5%E5%85%B7/Gradle.md)。

* gradle init --type java-library
	* 这里自动生成gradlew，并创建项目目录结构
* 之后所有命令使用gradlew即可

# gradle项目和maven项目相互转化
gradle和maven可以相互转化，意味着，我们可以使用gradle为主的开发，之后导出为maven项目，供生产环境使用。前提，你足够了解gradle和maven。

## maven -> gradle
* cd /path/to/mavenproject
* gradle init
* gradle wrapper

## gradle -> maven 
* cd /path/to/gradleproject
* gradlew install 

将项目转换为maven和gradle项目后，目录结构如下：
![dir_frame.png](/media/img/Java/gradle_maven/dir_frame.png)
之后，我们习惯使用mavnw或者gradlew，都可以。如此，做到了共存。

# 一个项目同时支持maven和gradle配置：一个好的开始
抽时间，做了常用jar包和插件整合包，一个项目同时支持maven和gradle。

**共同的依赖**：

* 内容包括： 日志、通用工具库、单元测试、代码质量度量、文档生成等
* jar: slf4j、logback、lombok、guava、junit、mockito

**配置中整合的工具**：

* 代码质量分析报告工具：pmd、findbugs、checkstyle、jdepend
* 单元测试报告工具、javadoc、依赖管理、项目信息汇总等可视化信息

**maven具体内容**

* maven-compiler-plugin、maven-javadoc-plugin、cobertura-maven-plugin、maven-checkstyle-plugin、findbugs-maven-plugin、maven-pmd-plugin、jdepend-maven-plugin、maven-jar-plugin、maven-surefire-plugin、maven-surefire-report-plugin

**gradle具体内容**

* java、maven、checkstyle、pmd、findbugs、jdepend、eclipse、idea、javadoc

首先maven配置见[此文件](https://github.com/ExFly/CsLearning/blob/master/NoteBookForDevelop/%E6%96%87%E6%A1%A3/Java/%E5%B7%A5%E5%85%B7/pom.common.xml)

其次gradle配置见[此文件](https://github.com/ExFly/CsLearning/blob/master/NoteBookForDevelop/%E6%96%87%E6%A1%A3/Java/%E5%B7%A5%E5%85%B7/build.common.gradle)

# 资料汇总
* [完整的整合项目，支持maven和gradle，点我下载](https://raw.githubusercontent.com/ExFly/CsLearning/master/NoteBookForDevelop/%E6%96%87%E6%A1%A3/Java/%E5%B7%A5%E5%85%B7/LombokL.zip)
* [我的Gradle笔记，点我查看](https://github.com/ExFly/CsLearning/blob/master/NoteBookForDevelop/%E6%96%87%E6%A1%A3/Java/%E5%B7%A5%E5%85%B7/Gradle.md)
* [我的maven笔记，点我查看](https://github.com/ExFly/CsLearning/blob/master/NoteBookForDevelop/%E6%96%87%E6%A1%A3/Java/%E5%B7%A5%E5%85%B7/Maven.md)
