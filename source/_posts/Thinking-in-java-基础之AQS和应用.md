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
> 排队接水喝，我的杯子放在这里，当前这个饮水机就我能用。其它人来了，不好意思，排队。

至于怎么抢占，又分为公平锁，和非公平锁，公平锁：先来后到，非公平：随机获取。

> 排队接水喝， 先后顺序，公平，来了个领导，他先接水，不公平，但合理。

AbstractOwnableSynchronizer

![图](Thinking-in-java-基础之AQS和应用/state.png)
CLH锁也是一种基于链表的可扩展、高性能、公平的自旋锁，申请线程仅仅在本地变量上自旋，它不断轮询前驱的状态，假设发现前驱释放了锁就结束自旋。

资源state使用volatile标识，保证所有的线程来的时候都是确定的值。详细查看[volatile]()，其它线程在队列里。

protected boolean tryAcquire(int arg) : 独占式获取同步状态，试着获取，成功返回true，反之为false
protected boolean tryRelease(int arg) ：独占式释放同步状态，等待中的其他线程此时将有机会获取到同步状态；
protected int tryAcquireShared(int arg) ：共享式获取同步状态，返回值大于等于0，代表获取成功；反之获取失败；
protected boolean tryReleaseShared(int arg) ：共享式释放同步状态，成功为true，失败为false
protected boolean isHeldExclusively() ： 是否在独占模式下被线程占用。


## CountDownLatch 倒计时器

它允许一个或多个线程一直等待，直到其他线程的操作执行完后再执行

CountDownLatch，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS(Compare and Swap)减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。
## CyclicBarrier 循环栅栏

让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。

> CountDownLatch: A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes. (CountDownLatch: 一个或者多个线程，等待其他多个线程完成某件事情之后才能执行；) CyclicBarrier : A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point.(CyclicBarrier : 多个线程互相等待，直到到达同一个同步点，再继续一起执行。)

对于CountDownLatch来说，重点是“一个线程（多个线程）等待”，而其他的N个线程在完成“某件事情”之后，可以终止，也可以等待。而对于CyclicBarrier，重点是多个线程，在任意一个线程没有完成，所有的线程都必须等待。

## Semaphore 信号量 

synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源,可以指定多个线程同时访问某个资源


## ReentrantLock 重入锁

> 排队接水喝，水杯太大，杯子放着接，我先干别的事，干完了回来，直接就能拿杯子，重入。

ReentrantLock，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

## tomcat的 LimitLatch 

- [Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)
- [Java并发包基石-AQS详解](https://www.cnblogs.com/chengxiao/archive/2017/07/24/7141160.html)
- [JavaGuide](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/AQS.md#41-countdownlatch-%E7%9A%84%E4%B8%89%E7%A7%8D%E5%85%B8%E5%9E%8B%E7%94%A8%E6%B3%95)