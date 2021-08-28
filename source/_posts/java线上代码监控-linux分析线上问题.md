---
title: java线上代码监控-linux分析线上问题
date: 2020-04-29 16:41:41
tags:
categories: java
description: 总结CPU怎么处理load 怎么处理
---

jps(Java Virtual Machine Process Status Tool)
jstack(查看线程)、jmap(查看内存)和jstat(性能分析)

### jps

jps 找到对应的进程ID

`jps -m -l `


```
-q 不输出类名、Jar名和传入main方法的参数
-m 输出传入main方法的参数
-l 输出main类或Jar的全限名
-v 输出传入JVM的参数
```
`ll /proc/[pid]/ | grep exe`

可以查到时哪一个后台进程在执行

## 未重启的话，分析现在的时候:

### jstack 线程问题

ps -mp [pid] -o THREAD,tid,time |sort -rn 打印CPU最高的线程
top -Hp [pid] 

printf "%x\n" 22338 把线程ID转换成16进制
5742

jstack [pid] | grep 5742 -A 30 打印堆栈信息


-l long listings，会打印出额外的锁信息，在发生死锁时可以用jstack -l pid来观察锁持有情况
-m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）

-F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
-m  to print both java and native frames (mixed mode)
-l  long listing. Prints additional information about locks
-h or -help to print this help message

[jstack Dump 日志文件中的线程状态](https://www.cnblogs.com/zhengyun_ustc/archive/2013/01/06/dumpanalysis.html)


线程死锁：
jstack –l pid | grep -i –E 'BLOCKED | deadlock'


### jmap 堆问题

jmap -heap [pid] 先查看堆栈

```
[root@localhost bin]# ./jmap -heap 22327
Attaching to process ID 22327, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.201-b09

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 2147483648 (2048.0MB)
   NewSize                  = 89128960 (85.0MB)
   MaxNewSize               = 715653120 (682.5MB)
   OldSize                  = 179306496 (171.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 671612928 (640.5MB)
   used     = 349908376 (333.6986312866211MB)
   free     = 321704552 (306.8013687133789MB)
   52.0997082414709% used
From Space:
   capacity = 12582912 (12.0MB)
   used     = 12566640 (11.984481811523438MB)
   free     = 16272 (0.0155181884765625MB)
   99.87068176269531% used
To Space:
   capacity = 22020096 (21.0MB)
   used     = 0 (0.0MB)
   free     = 22020096 (21.0MB)
   0.0% used
PS Old Generation
   capacity = 236978176 (226.0MB)
   used     = 45630600 (43.51673126220703MB)
   free     = 191347576 (182.48326873779297MB)
   19.255190823985412% used

38073 interned Strings occupying 3725472 bytes.
```
查看对象

用jmap -histo[:live] pid查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象，如下：

数量和容量

实例数量前十的类：jmap -histo pid | sort -n -r -k 2 | head -10

实例容量前十的类：jmap -histo pid | sort -n -r -k 3 | head -10


#### 生成hprof文件（hprof（Heap/CPU Profiling Tool））

`./jmap -dump:live,format=b,file=dump.dump [pid]`

可以用：`./jhat -port 9998 dump.dump`  `./jhat -J-Xmx512m -port 9998 dump.dump`
jhat -port 7401 -J -Xmx4G dump.dump

或者dump文件修改为dump.hprof,可以加到jprofiler

作用：主要分析内部当时的对象。

### jstat

jstat（JVM统计监测工具）

每隔250ms采集4次`./jstat -gc 30359 250 4`
S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
区容量（Capacity）和使用量（Used）
S0 S1 Survivor E O Eden Old M MetaspaceSize CCS CompressedClassSpaceSize  YG young gc FG  full gc T time FCT full gc time

`./jstat -options` 参数 -gc  

250ms调用4次

```
[root@localhost bin]# ./jstat -gc 22327  250 4
 S0C    S1C    S0U    S1U      EC       EU        OC         OU            MC     MU     CCSC   CCSU      YGC     YGCT    FGC    FGCT     GCT   
12288.0 21504.0 12272.1  0.0   655872.0 341707.4  231424.0   44561.1   87384.0 83940.7 11352.0 10729.1     18    0.238   3      0.254    0.492
12288.0 21504.0 12272.1  0.0   655872.0 341707.4  231424.0   44561.1   87384.0 83940.7 11352.0 10729.1     18    0.238   3      0.254    0.492
12288.0 21504.0 12272.1  0.0   655872.0 341707.4  231424.0   44561.1   87384.0 83940.7 11352.0 10729.1     18    0.238   3      0.254    0.492
12288.0 21504.0 12272.1  0.0   655872.0 341707.4  231424.0   44561.1   87384.0 83940.7 11352.0 10729.1     18    0.238   3      0.254    0.492
```


## 准备重启复现的话：


第一步 加参数 ：

gc日志 配置Catalina.bat JAVA_OPTS

`-Xloggc:./logs/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=32  -XX:GCLogFileSize=64m -XX:+HeapDumpOnOutOfMemoryError `

- [步骤:线上问题定位，重启大法好！线上常见问题排查手册](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247495546&idx=1&sn=f435f56c3b92802ef649cd14bb60711b&chksm=e92ad275de5d5b63b9fc1e48463044ea1b234a4f28d0ad8d7964c42516f1cf7cae47c60162dc&scene=0&xtrack=1&exportkey=AeNCSm6QS4EwNcRCTNfXDUs%3D&pass_ticket=sHf4d8XsExleK1yRet7Gq5oJd6I%2FDOYRgopyFK8gH%2B81%2FhlGHzeiSrSlgR9oXDlk#rd)
- [VM性能调优监控工具jps、jstack、jmap、jhat、jstat、hprof使用详解](https://blog.csdn.net/muyimo/article/details/52837323?utm_source=distribute.pc_relevant.none-task-blog-baidujs-8)


一  线上常见问题定位

常见问题 1：CPU 利用率高

CPU 使用率是衡量系统繁忙程度的重要指标，一般情况下单纯的 CPU 高并没有问题，它代表系统正在不断的处理我们的任务，但是如果 CPU 过高，导致任务处理不过来，从而引起 load 高，这个是非常危险需要关注的。 CPU 使用率的安全值没有一个标准值，取决于你的系统是计算密集型还是 IO 密集型，一般计算密集型应用 CPU 使用率偏高 load 偏低，IO 密集型相反。

问题原因及定位：

1  频繁 FullGC/YongGC

2  代码消耗，如死循环，md5 等内存态操作


常见问题 2：load 高

load 指单位时间内活跃进程数，包含运行态（runnable 和 running）和不可中断态（ IO、内核态锁）。关键字是运行态和不可中断态，运行态可以联想到 Java 线程的 6 种状态，如下，线程 new 之后处于 NEW 状态，执行 start 进入 runnable 等待 CPU 调度，因此如果 CPU 很忙会导致 runnable 进程数增加；不可中断态主要包含网络 IO、磁盘 IO 以及内核态的锁，如 synchronized 等。

问题原因及定位：

1  CPU 利用率高，可运行态进程数多

2  iowait，等待 IO

3  等待内核态锁，如 synchronized


常见问题 3：持续 FullGC

问题原因及定位：

1  prommotion failed

从S区晋升的对象在老年代也放不下导致 FullGC（fgc 回收无效则抛 OOM）。

1）survivor 区太小，对象过早进入老年代。

2）大对象分配，没有足够的内存。

日志查找关键字 “allocating large”；

2  concurrent mode failed

在 CMS GC 过程中业务线程将对象放入老年代（并发收集的特点）内存不足。详细原因：

1）fgc 触发比例过大，导致老年代占用过多，并发收集时用户线程持续产生对象导致达到触发 FGC 比例。

jinfo  查看 CMSInitiatingOccupancyFraction 参数，一般 70~80 即可

2）老年代存在内存碎片。

jinfo  查看 UseCMSCompactAtFullCollection 参数，在 FullGC 后整理内存

常见问题 4：线程池满

Java 线程池以有界队列的线程池为例，当新任务提交时，如果运行的线程少于 corePoolSize，则创建新线程来处理请求。如果正在运行的线程数等于 corePoolSize 时，则新任务被添加到队列中，直到队列满。当队列满了后，会继续开辟新线程来处理任务，但不超过 maximumPoolSize。当任务队列满了并且已开辟了最大线程数，此时又来了新任务，ThreadPoolExecutor 会拒绝服务。

问题原因及定位：

2  数据库慢 sql 或者数据库死锁

日志关键字 “Deadlock found when trying to get lock”

Jstack 或 zprofiler 查看阻塞态线程

3  Java 代码死锁

jstack –l pid | grep -i -E 'BLOCKED | deadlock'
