---
title: Thinking in java 高级之对象大小模型
date: 2020-05-19 10:50:29
tags:
categories: java
description: "Java对象模型，对象大小，对象头，数组对象，如何计算，怎么验证，(OOP)Ordinary Object Pointer"
---


## JOL工具

## 大小

### _markword

markword 
第一部分markword,用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，官方称它为“MarkWord”。
klass 
对象头的另外一部分是klass类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例. 32位4字节，64位开启指针压缩或最大堆内存<32g时 4字节，否则8字节
数组长度（只有数组对象有） 4字节
如果对象是一个数组, 那在对象头中还必须有一块数据用于记录数组长度.int最大值2g，2^31，java数组（包含字符串）最长2g

### KlassOop

### 实际数据大小

#### 基本类型
#### 对象类型
#### 数组类型

## 计算
### 本身大小
### ShallowSize 
