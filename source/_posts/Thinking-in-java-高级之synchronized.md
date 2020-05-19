---
title: Thinking in java 高级之synchronized
date: 2020-05-07 16:39:25
tags:
categories: java
description: 从synchronized到汇编,结合锁升级,探究背后的本质，
---

### 先从内核态和用户态说起
0x80 软中断 频繁切换会耗时

### 从DCL到锁升级
Double Check Lock 单例

普通 偏向锁 轻量级锁 重量级锁

升级过程

### 需要_markword信息 标记有哪一种锁

### 
### 



[Synchronized解析——如果你愿意一层一层剥开我的心](https://juejin.im/post/5d5374076fb9a06ac76da894#heading-18)
[【转载】Java中的锁机制 synchronized & 偏向锁 & 轻量级锁 & 重量级锁 & 各自优缺点及场景 & AtomicReference](https://www.cnblogs.com/charlesblc/p/5994162.html)