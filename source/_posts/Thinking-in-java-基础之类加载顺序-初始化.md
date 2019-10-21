---
title: Thinking in java 基础之类加载顺序&初始化
date: 2019-10-21 15:25:42
tags:
categories: java
description: "类加载过程"
---


#### 先来一个题

![初始化](Thinking-in-java-基础之类加载顺序-初始化/Class初始化.png)

#### 初始化

##### java对象，获取
- new 
- 反射
- 反序列化
- 克隆

##### Class对象获取

- 类名.class
- 实例.getClass()
- classloader.loadClass("包.类")
- Class.forName("包.类");

![初始化](Thinking-in-java-基础之类加载顺序-初始化/获取Class四种方式.PNG)



![初始化](Thinking-in-java-基础之类加载顺序-初始化/Class初始化时机.png)

#### 
