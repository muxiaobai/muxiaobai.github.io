---
title: java线上代码监控-工具(汇总)
date: 2019-08-16 13:32:13
tags:
categories: java
description: "代码线上检测,快速查找bug"
---

强力推荐alibaba的arthas

## arthas 

官网[https://alibaba.github.io/arthas](https://alibaba.github.io/arthas)

教程[Arthas 基础教程](https://alibaba.github.io/arthas/arthas-tutorials?language=cn&id=arthas-basics)

教程[Arthas 进阶教程](https://alibaba.github.io/arthas/arthas-tutorials?language=cn&id=arthas-advanced)

启动

```
wget https://alibaba.github.io/arthas/arthas-boot.jar
java -jar arthas-boot.jar --target-ip 0.0.0.0
```
选择对应的main

通用命令
- keymap 快捷键
- help
- Q ctrl+c 退出当前命令

常用命令
- dashboard 实时看板
- thread 查线程
- jvm 当前jvm信息
- sysprop(sysenv) 系统参数(System Property)(System Environment Variables)
- jad 反编译
- sc (search class)查jvm加载的类 -d 加载的信息 classLoaderHash 哪一个ClassLoader加载的类
- sm (search method) 查class方法
- mc (Memory Compiler) 使用指定的ClassLoader编译java
- redefine 热更新代码
- watch 方法执行数据观测（实时调用）
- ognl 动态执行代码

```
反编译 修改 编译 热更新
jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java
vim /tmp/UserController.java
mc /tmp/UserController.java -d /tmp 
redefine /tmp/com/example/demo/arthas/user/UserController.class
```


调用请求: curl -w "@curl-time.txt" http://127.0.0.1:8080/hello

## 常用和spring有关的

从spring-mvc入手


### 内部：
trace 方法内部调用路径，并输出方法路径上的每个节点上耗时 -j跳过jdkclass
Servlet Filter 转发 哪一个Controller

trace javax.servlet.Servlet *
trace javax.servlet.Filter *
trace org.springframework.web.servlet.DispatcherServlet *

watch org.springframework.web.servlet.DispatcherServlet getHandler returnObj
是哪一个controller调用

外部：
stack org.springframework.web.servlet.DispatcherServlet *
stack 输出当前方法被调用的调用路径
很多时候我们都知道一个方法被执行，但这个方法被执行的路径非常多，或者你根本就不知道这个方法是从那里被执行了，此时你需要的是 stack 命令。

## 其它监控

Druid连接池 数据库监控 参考 [数据库监控-Druid监控配置](https://muxiaobai.github.io/2018/11/22/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9B%91%E6%8E%A7-Druid%E7%9B%91%E6%8E%A7%E9%85%8D%E7%BD%AE/)
Javamelody 传统应用 参考 [Javamelody 工具](https://muxiaobai.github.io/2017/08/02/Javamelody-%E5%B7%A5%E5%85%B7/)
jre自带的工具 jconsole jvisualvm参考 [深入理解JVM之远程监控](https://muxiaobai.github.io/2018/11/22/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3JVM%E4%B9%8B%E8%BF%9C%E7%A8%8B%E7%9B%91%E6%8E%A7/)


