---
title: java线上代码监控-工具-arthas
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

   参数：--tunnel-server 'ws://47.75.156.201:7777/ws'
默认的是
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
- classloader
- sysprop(sysenv) 系统参数(System Property)(System Environment Variables)
- jad 反编译
- stack 调用栈
- trace 调用时间
- watch 方法执行数据观测（实时调用）
- sc (search class)查jvm加载的类 -d 加载的信息 classLoaderHash 哪一个ClassLoader加载的类
- sm (search method) 查class方法
- mc (Memory Compiler) -c 使用指定的ClassLoader编译java javac编译
- redefine 热更新代码
- ognl 动态执行代码
- tt 可以记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

```
反编译 修改 编译 热更新
jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java
vim /tmp/UserController.java
sc com.example.demo.arthas.user.UserController -d 
mc /tmp/UserController.java -d /tmp -c [classLoaderHash]
redefine /tmp/com/example/demo/arthas/user/UserController.class
```

实际上可以直接把本机的class redefine 线上的很可能编译不通过，不过要保证本机代码和线上的代码同步

### 暂存运行状态文件hpof

类似jmap命令的heap dump功能。

```
heapdump /tmp/dump.hprof

```

只dump live对象

```

[arthas@58205]$ heapdump --live /tmp/dump.hprof
Dumping heap to /tmp/dump.hprof...
Heap dump file created

```

分析hprof文件工具
- 使用eclipse的MAT (Memory Analyzer Tools)
- 或者IDEA 的jprofiler插件 [IDEA 配置执行器](https://blog.csdn.net/wytocsdn/article/details/79258247) 需要装jprofiler执行器[jprofiler下载](https://www.ej-technologies.com/download/jprofiler/version_92) [license](https://www.cnblogs.com/jifeng/p/3336408.html)  [jprofiler 使用](https://blog.csdn.net/vicky_pyh/article/details/88797514)
- IMB 的heapAnalyzer

```

按默认选择“Single or evaluation license”
Name 和 Company 随意
-----------------------忧郁的分割线---------------------------
L-Larry_Lau@163.com#23874-hrwpdp1sh1wrn#0620
L-Larry_Lau@163.com#36573-fdkscp15axjj6#25257
L-Larry_Lau@163.com#5481-ucjn4a16rvd98#6038
L-Larry_Lau@163.com#99016-hli5ay1ylizjj#27215
L-Larry_Lau@163.com#40775-3wle0g1uin5c1#0674
--------------------------------------------------------------
L-Larry_Lau@163.com#7009-14frku31ynzpfr#20176
L-Larry_Lau@163.com#49604-1jfe58we9gyb6#5814
L-Larry_Lau@163.com#25531-1qcev4yintqkj#23927
L-Larry_Lau@163.com#96496-1qsu1lb1jz7g8w#23479
L-Larry_Lau@163.com#20948-11amlvg181cw0p#171159

```

JRebel和JProfiler同时运行

如果IDEA中配置了JRebel热部署，要想JRebel和JProfiler同时运行，就要改手动管理JProfiler session了。
在 Run–Debug Configurations窗口JRebel Debug tab页中配置Environment Variables属性 CATALINA_OPTS：
使用springboot不成功，
CATALINA_OPTS=-agentpath:E:\tools\jprofiler_windows-x64_9_2_1\jprofiler9\bin\windows-x64\jprofilerti.dll=port=8849,nowait,id=81,config=C:\Users\Administrator\.jprofiler9\config.xml

调用请求: curl -w "@curl-time.txt" http://127.0.0.1:8080/hello

###常用

thread -n 5
2）查看某个函数的调用堆栈 打印前五名最消耗CPU的线程，可以及时找到CPU过高的代码位置

stack <类全包名> <函数名> 往来的地方查找 调用栈
3）查看某个函数的哪个子调用最慢，耗时最久的调用会标红显示，可以方便找出某个功能中最耗时的操作

trace <类全包名> <函数名> 继续往下查找 追溯追踪
4）监控某个函数的调用统计数据，包括总调用次数，平均运行时间，成功率等信息

monitor <类全包名> <函数名>

###  热更新代码


[热更新](https://mp.weixin.qq.com/s/um6XQVv1JvHM3n7NslzB0Q)

## 常用和spring有关的

从spring-mvc入手


### 内部：
trace 方法内部调用路径，并输出方法路径上的每个节点上耗时 -j跳过jdkclass
Servlet Filter 转发 哪一个Controller

trace javax.servlet.Servlet *
trace javax.servlet.Filter *
trace org.springframework.web.servlet.DispatcherServlet *
```
[arthas@31978]$ trace  org.springframework.web.servlet.DispatcherServlet *
Press Q or Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:48) cost in 214 ms.
`---ts=2020-05-07 11:28:07;thread_name=http-nio-18201-exec-5;id=40;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@73a2e526
    `---[196.804639ms] org.springframework.web.servlet.DispatcherServlet:doService()
        +---[0.025295ms] org.springframework.web.servlet.DispatcherServlet:logRequest() #910
        |   `---[0.013937ms] org.springframework.web.servlet.DispatcherServlet:logRequest()
        |       `---[0.001864ms] org.springframework.core.log.LogFormatUtils:traceDebug() #955
        +---[0.001121ms] org.springframework.web.util.WebUtils:isIncludeRequest() #915
        +---[7.43E-4ms] org.springframework.web.servlet.DispatcherServlet:getWebApplicationContext() #927
        +---[0.001808ms] javax.servlet.http.HttpServletRequest:setAttribute() #927
        +---[5.21E-4ms] javax.servlet.http.HttpServletRequest:setAttribute() #928
        +---[6.67E-4ms] javax.servlet.http.HttpServletRequest:setAttribute() #929
        +---[0.017824ms] org.springframework.web.servlet.DispatcherServlet:getThemeSource() #930
        |   `---[0.009355ms] org.springframework.web.servlet.DispatcherServlet:getThemeSource()
        |       `---[min=3.26E-4ms,max=4.25E-4ms,total=7.51E-4ms,count=2] org.springframework.web.servlet.DispatcherServlet:getWebApplicationContext() #803
        +---[8.34E-4ms] javax.servlet.http.HttpServletRequest:setAttribute() #930
        +---[0.002303ms] org.springframework.web.servlet.FlashMapManager:retrieveAndUpdate() #933
        +---[0.001279ms] org.springframework.web.servlet.FlashMap:<init>() #937
        +---[5.06E-4ms] javax.servlet.http.HttpServletRequest:setAttribute() #937
        +---[4.63E-4ms] javax.servlet.http.HttpServletRequest:setAttribute() #938
        +---[196.6798ms] org.springframework.web.servlet.DispatcherServlet:doDispatch() #942 这里时间最长196ms doDispatch
        |   `---[196.653121ms] org.springframework.web.servlet.DispatcherServlet:doDispatch()
        |       +---[0.001231ms] org.springframework.web.context.request.async.WebAsyncUtils:getAsyncManager() #1004
        |       +---[0.074447ms] org.springframework.web.servlet.DispatcherServlet:checkMultipart() #1011
        |       |   `---[0.049658ms] org.springframework.web.servlet.DispatcherServlet:checkMultipart()
        |       |       `---[0.03953ms] org.springframework.web.multipart.MultipartResolver:isMultipart() #1164
        |       +---[0.095085ms] org.springframework.web.servlet.DispatcherServlet:getHandler() #1015
        |       |   `---[0.081615ms] org.springframework.web.servlet.DispatcherServlet:getHandler()
        |       |       `---[min=0.010315ms,max=0.056882ms,total=0.067197ms,count=2] org.springframework.web.servlet.HandlerMapping:getHandler() #1232
        |       +---[7.93E-4ms] org.springframework.web.servlet.HandlerExecutionChain:getHandler() #1022
        |       +---[0.02294ms] org.springframework.web.servlet.DispatcherServlet:getHandlerAdapter() #1022
        |       |   `---[0.010588ms] org.springframework.web.servlet.DispatcherServlet:getHandlerAdapter()
        |       |       `---[0.00129ms] org.springframework.web.servlet.HandlerAdapter:supports() #1268
        |       +---[8.17E-4ms] javax.servlet.http.HttpServletRequest:getMethod() #1025
        |       +---[6.104335ms] org.springframework.web.servlet.HandlerExecutionChain:applyPreHandle() #1034
        |       +---[7.69E-4ms] org.springframework.web.servlet.HandlerExecutionChain:getHandler() #1039
        |       +---[190.13169ms] org.springframework.web.servlet.HandlerAdapter:handle() #1039   最终落在handle中
        |       +---[0.001149ms] org.springframework.web.context.request.async.WebAsyncManager:isConcurrentHandlingStarted() #1041
        |       +---[0.087605ms] org.springframework.web.servlet.DispatcherServlet:applyDefaultViewName() #1045
        |       |   `---[0.019858ms] org.springframework.web.servlet.DispatcherServlet:applyDefaultViewName()
        |       +---[0.004415ms] org.springframework.web.servlet.HandlerExecutionChain:applyPostHandle() #1046
        |       +---[0.075878ms] org.springframework.web.servlet.DispatcherServlet:processDispatchResult() #1056
        |       |   `---[0.034479ms] org.springframework.web.servlet.DispatcherServlet:processDispatchResult()
        |       |       +---[0.001589ms] org.apache.commons.logging.Log:isTraceEnabled() #1123
        |       |       +---[0.002227ms] org.springframework.web.context.request.async.WebAsyncUtils:getAsyncManager() #1128
        |       |       +---[4.4E-4ms] org.springframework.web.context.request.async.WebAsyncManager:isConcurrentHandlingStarted() #1128
        |       |       `---[0.002093ms] org.springframework.web.servlet.HandlerExecutionChain:triggerAfterCompletion() #1134
        |       `---[3.93E-4ms] org.springframework.web.context.request.async.WebAsyncManager:isConcurrentHandlingStarted() #1066
        +---[7.31E-4ms] org.springframework.web.context.request.async.WebAsyncUtils:getAsyncManager() #945
        `---[3.94E-4ms] org.springframework.web.context.request.async.WebAsyncManager:isConcurrentHandlingStarted() #945

```

watch org.springframework.web.servlet.DispatcherServlet getHandler returnObj
是哪一个controller调用

### 外部：
stack org.springframework.web.servlet.DispatcherServlet *
stack 输出当前方法被调用的调用路径
很多时候我们都知道一个方法被执行，但这个方法被执行的路径非常多，或者你根本就不知道这个方法是从那里被执行了，此时你需要的是 stack 命令。

## 其它监控

- Druid连接池 数据库监控 参考 [数据库监控-Druid监控配置](https://muxiaobai.github.io/2018/11/22/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9B%91%E6%8E%A7-Druid%E7%9B%91%E6%8E%A7%E9%85%8D%E7%BD%AE/)
- Javamelody 传统应用 参考 [Javamelody 工具](https://muxiaobai.github.io/2017/08/02/Javamelody-%E5%B7%A5%E5%85%B7/)
- jre自带的工具 jconsole jvisualvm参考 [深入理解JVM之远程监控](https://muxiaobai.github.io/2018/11/22/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3JVM%E4%B9%8B%E8%BF%9C%E7%A8%8B%E7%9B%91%E6%8E%A7/)


![参考](java线上代码监控-工具-arthas/arthas.png)