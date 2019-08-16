---
title: hadoop-Sqoop-Hive等的安装
date: 2017-07-20 06:38:48
tags:   
categories: [hadoop]
description: "简单介绍安装过程，以及各个工具的作用"
---

先介绍工具，然后进行安装
#### hadoop 大象

HDFS 文件系统 存储
MapReduce     计算
YARN 资源分配、管理工具MapReducev2
HBase

[hadoop的安装](https://muxiaobai.github.io/2017/07/19/Linux-%E4%B8%8B%E7%9A%84java%E5%AE%89%E8%A3%85%E3%80%81%E5%8F%98%E6%9B%B4%E4%BB%A5%E5%8F%8A%E5%8D%87%E7%BA%A7%E9%97%AE%E9%A2%98/)


#### sqoop

一个Apache的命令行工具 ，作用是：Transformation

hadoop(HBase,Hive) 和结构化数据(structural data)的转换
1.4.6 和1.99.3 建议1.4.6更稳定，使用的广泛，常见问题大都有解决。
[Sqoop 1.99.3 安装](http://houshangxiao.iteye.com/blog/2070057)

#### Hive 小蜜蜂 和 Pig 猪
数据仓库
MR写起来比较繁琐，产生Hive，而且Hive比较易懂，方便。
一种通过SQL来操作分布式存储数据的数据仓库工具。交换式查询
FaceBook开发的 
文件类型: ',' 'tab' 本本文件  ORC  Parquet
访问方式: CLI JDBC/ODBC WEB UI  Derby 内存数据库 单用户连接
Impala 是实时的

Hive 默认是 MapReduce 引擎
Hive on Spark

#### Spark

Spark MapReduce Storm 是三种不同的计算处理
MapReduce 是大的批量操作，不要求时限。基于文件系统，hdfs。
Spark 是快速的批量操作，基于内存，所以速度快。其主要亮点在于把过程给数据。
Storm是流式处理，快速实时。
Spark Streaming 跟Storm类似，只不过Spark Streaming是小时间窗口的处理，Storm是实时的来一条处理一条。
[三者比较](http://blog.csdn.net/wangfei8348/article/details/52924723)
Spark SQL


#### Zookeeper 分布式协调服务

在此基础上 加上netty alibaba开发出了Dubbo服务治理。










