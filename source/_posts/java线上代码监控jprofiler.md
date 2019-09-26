---
title: java线上代码监控jprofiler
date: 2019-09-26 09:45:43
tags:
categories: java
description: "代码线上检测,快速查找bug"
---


启动一个jvm进程

java -jar xxx.0.0.1.jar


服务器解压[服务器端](https://download-gcdn.ej-technologies.com/jprofiler/jprofiler_linux_11_0_2.tar.gz)
[window客户端](https://download-gcdn.ej-technologies.com/jprofiler/jprofiler_windows-x64_11_0_2.zip)

服务器

- 解压服务器
- 启动./jprofiler11.0.2/bin.jpenable
- 选择要监控功能的jvm
- 选择使用远程GUI监控
- 设置端口

客户端

新建一个remote Session

