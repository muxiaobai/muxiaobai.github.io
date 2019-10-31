---
title: Thinking in java 基础之AQS和应用
date: 2019-10-31 13:12:44
tags: Thread
categories: java
description: "jdk的juc中的AQS和应用，各种锁的比较"
---

使用锁的时候我们直接`ReentrantLock lock = new ReentrantLock();lock.lock();`,这中间究竟有什么内容,
lock的时候，其它线程为什么不能执行，unlook()的时候其它的线程究竟从哪个线程能执行？本文主要分析lock的各种情况。

## j.u.c.locks.AbstractQueuedSynchronizer

有共享和独占两种状态，SHARE，EXCLUSIVE 这两个在AQS中对应两个Node节点{state:1,thread:Thread.CurrentThread()},节点里面的内容。
从名字里面就能知道，共享锁，每一个线程来都可以拿到锁，简单的来说，就是可以运行lock和unlock内的代码，独占锁，一个线程lock，其它线程必须等到unlock之后，再去抢占锁。

至于怎么抢占，又分为公平锁，和非公平锁，公平锁：先来后到，非公平：随机获取。

AbstractOwnableSynchronizer

![图](Thinking-in-java-基础之AQS和应用/state.png)
资源state使用volatile标识，保证所有的线程来的时候都是确定的值。详细查看[volatile]()，其它线程在队列里。

## Semaphore 信号量 

## CountDownLatch 

## CyclicBarrier 

## ReentrantLock 重入锁

## tomcat的 LimitLatch 

- [Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)
- [Java并发包基石-AQS详解](https://www.cnblogs.com/chengxiao/archive/2017/07/24/7141160.html)
- [JavaGuide](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/AQS.md#41-countdownlatch-%E7%9A%84%E4%B8%89%E7%A7%8D%E5%85%B8%E5%9E%8B%E7%94%A8%E6%B3%95)