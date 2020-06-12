---
title: Thinking in java 高级之synchronized
date: 2020-05-07 16:39:25
tags:
categories: java
description: 从synchronized到汇编,结合锁升级,探究背后的本质，
---

本文主要

### 先从内核态和用户态说起
0x80 软中断 频繁切换会耗时，切换到内核态，调用内核方法。

### 从DCL到锁升级
Double Check Lock 单例[参考Singleton.java](https://github.com/muxiaobai/java-demo/blob/master/test-java-demo/src/main/java/io/github/muxiaobai/java/java/threadTest/Singleton.java)


普通 偏向锁 轻量级锁 重量级锁


升级过程


### 需要_markword信息 标记有哪一种锁
![java 对象大小](Thinking-in-java-高级之synchronized/锁偏向.png)

### java字节码指令
```
  private static sync classSync = new sync();
 public void doSth(){
        synchronized (sync.class){
            System.out.println("test Synchronized 锁的是 class对象 因此this不会标记" );
            System.out.println(ClassLayout.parseInstance(this).toPrintable());
        }
        synchronized (classSync){
            System.out.println("test Synchronized 锁的是 private classSync对象 因此this会标记" );
            System.out.println(ClassLayout.parseInstance(this).toPrintable());
        }
    }
```

sync锁对象，一个是锁class对象，一个是锁object对象。
```
 0 ldc #2 <io/github/muxiaobai/java/java/threadTest/base/sync>
 2 dup
 3 astore_1
 4 monitorenter   sync进入
 5 getstatic #6 <java/lang/System.out>
 8 ldc #12 <test Synchronized 锁的是 class对象 因此this不会标记>
10 invokevirtual #8 <java/io/PrintStream.println>
13 getstatic #6 <java/lang/System.out>
16 aload_0
17 invokestatic #10 <org/openjdk/jol/info/ClassLayout.parseInstance>
20 invokevirtual #11 <org/openjdk/jol/info/ClassLayout.toPrintable>
23 invokevirtual #8 <java/io/PrintStream.println>
26 aload_1
27 monitorexit   sync退出
28 goto 36 (+8)  异常退出 跳到36行，aload_0 继续往下执行，下一个sync
31 astore_2
32 aload_1
33 monitorexit    sync退出
34 aload_2
35 athrow
36 aload_0
37 getfield #4 <io/github/muxiaobai/java/java/threadTest/base/sync.classSync>
40 dup
41 astore_1
42 monitorenter  sync进入
43 getstatic #6 <java/lang/System.out>
46 ldc #13 <test Synchronized 锁的是 private classSync对象 因此this会标记>
48 invokevirtual #8 <java/io/PrintStream.println>
51 getstatic #6 <java/lang/System.out>
54 aload_0
55 invokestatic #10 <org/openjdk/jol/info/ClassLayout.parseInstance>
58 invokevirtual #11 <org/openjdk/jol/info/ClassLayout.toPrintable>
61 invokevirtual #8 <java/io/PrintStream.println>
64 aload_1
65 monitorexit   sync退出
66 goto 74 (+8)  异常直接跳到 74行return
69 astore_3
70 aload_1
71 monitorexit  sync退出
72 aload_3
73 athrow
74 return
```
注意 monitorenter 和monitorexit  ，每一个sync关键字，对应 一个enter和两个exit，正常退出和异常退出。

### JIT 
即时编译JIT只在代码段执行足够次数才会进行优化，在执行过程中不断收集各种数据，作为优化的决策; 什么代码会被即时编译 JIT，？

`lock cmpxchg`

方法上的锁调用方法的对象

```
 public synchronized void doSth1(){
        System.out.println("test Synchronized method，sync调用这个方法，方法上的，会锁this对象 ，也就是main方法中 new的sync" );
        System.out.println(ClassLayout.parseInstance(this).toPrintable());
    }
```

// access flags 0x21
public synchronized doSth1()V
```
 0 getstatic #6 <java/lang/System.out>
 3 ldc #14 <test Synchronized method，sync调用这个方法，方法上的，会锁this对象 ，也就是main方法中 new的sync>
 5 invokevirtual #8 <java/io/PrintStream.println>
 8 getstatic #6 <java/lang/System.out>
11 aload_0
12 invokestatic #10 <org/openjdk/jol/info/ClassLayout.parseInstance>
15 invokevirtual #11 <org/openjdk/jol/info/ClassLayout.toPrintable>
18 invokevirtual #8 <java/io/PrintStream.println>
21 return

```

### 



- [Synchronized解析——如果你愿意一层一层剥开我的心](https://juejin.im/post/5d5374076fb9a06ac76da894#heading-18)
- [【转载】Java中的锁机制 synchronized & 偏向锁 & 轻量级锁 & 重量级锁 & 各自优缺点及场景 & AtomicReference](https://www.cnblogs.com/charlesblc/p/5994162.html)
