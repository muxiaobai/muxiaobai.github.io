---
title: mongodb主从架构设计
date: 2019-01-11 02:49:11
tags: [mongo,设计]
categories: [数据库]
description: "设计mongo的主从架构"
---

基本的MongoDB主从搭建语句，汇总，以便之后查阅。
<!--more-->

带参数启动MongoDB：` ./mongod -f mongo-master.conf `


基本单机配置文件

```
# mongo.conf

# fork and run in background
fork=true

# 端口
port=9430

# 认证模式
auth=true

# what storage engine to use - defaults to wiredTiger if no data files present directory for datafiles - defaults to /data/db
storageEngine=wiredTiger

# maximum amount of memory to allocate for cache; defaults to 1/2 of physical RAM
wiredTigerCacheSizeGB=1

# 数据库文件保存位置
dbpath=/db/mongodb-3.4.7/data/db

# where to log
logpath=/db/logs/mongodb.log

#以追加方式写入日志
logappend=true

# each database will be stored in a separate directory
directoryperdb=true

# use a smaller default file size
smallfiles=true

# enable http interface， port=28017
httpinterface=false

# turn on simple rest api 开启rest api
rest=false

# comma separated list of ip addresses to  listen on - all local ips by default
# bind_ip=192.168.1.120,192.168.1.123

```

简单的主从配置文件

master.conf
```
# 设置数据文件的存放目录
dbpath=/db/mongo/master/data
# 设置日志文件的存放目录及其日志文件名
logpath=/db/mongo/master/logs/mongo.log
# 设置端口号（默认的端口号是 27017）
master=true
#slave=true
# 设置为以守护进程的方式运行，即在后台运行
fork=true
#监听网卡
bind_ip= 0.0.0.0
#服务端口
port=27019
oplogSize=2048
#source=127.0.0.1:27018
#autoresync=true

```

slave.conf

```
# 设置数据文件的存放目录
dbpath=/db/mongo/slave/data
# 设置日志文件的存放目录及其日志文件名
logpath=/db/mongo/slave/logs/mongo.log
# 设置端口号（默认的端口号是 27017）
#master=true
slave=true
# 设置为以守护进程的方式运行，即在后台运行
fork=true
#监听网卡
bind_ip= 0.0.0.0
#服务端口
port=27019
oplogSize=2048
source=127.0.0.1:27018
autoresync=true

```

一些其他的参数：
```
--quiet # 安静输出
--port arg  # 指定服务端口号，默认端口27017
--bind_ip arg   # 绑定服务IP，若绑定127.0.0.1，则只能本机访问，不指定默认本地所有IP
--logpath arg   # 指定MongoDB日志文件，注意是指定文件不是目录
--logappend # 使用追加的方式写日志
--pidfilepath arg   # PID File 的完整路径，如果没有设置，则没有PID文件
--keyFile arg   # 集群的私钥的完整路径，只对于Replica Set 架构有效
--unixSocketPrefix arg  # UNIX域套接字替代目录,(默认为 /tmp)
--fork  # 以守护进程的方式运行MongoDB，创建服务器进程
--auth  # 启用验证
--cpu   # 定期显示CPU的CPU利用率和iowait
--dbpath arg    # 指定数据库路径
--diaglog arg   # diaglog选项 0=off 1=W 2=R 3=both 7=W+some reads
--directoryperdb    # 设置每个数据库将被保存在一个单独的目录
--journal   # 启用日志选项，MongoDB的数据操作将会写入到journal文件夹的文件里
--journalOptions arg    # 启用日志诊断选项
--ipv6  # 启用IPv6选项
--jsonp # 允许JSONP形式通过HTTP访问（有安全影响）
--maxConns arg  # 最大同时连接数 默认2000
--noauth    # 不启用验证
--nohttpinterface   # 关闭http接口，默认关闭27018端口访问
--noprealloc    # 禁用数据文件预分配(往往影响性能)
--noscripting   # 禁用脚本引擎
--notablescan   # 不允许表扫描
--nounixsocket  # 禁用Unix套接字监听
--nssize arg (=16)  # 设置信数据库.ns文件大小(MB)
--objcheck  # 在收到客户数据,检查的有效性，
--profile arg   # 档案参数 0=off 1=slow, 2=all
--quota # 限制每个数据库的文件数，设置默认为8
--quotaFiles arg    # number of files allower per db, requires --quota
--rest  # 开启简单的rest API
--repair    # 修复所有数据库run repair on all dbs
--repairpath arg    # 修复库生成的文件的目录,默认为目录名称dbpath
--slowms arg (=100) # value of slow for profile and console log
--smallfiles    # 使用较小的默认文件
--syncdelay arg (=60)   # 数据写入磁盘的时间秒数(0=never,不推荐)
--sysinfo   # 打印一些诊断系统信息
--upgrade   # 如果需要升级数据库
 * Replicaton 参数
 
--------------------------------------------------------------------------------
 
 
--fastsync  # 从一个dbpath里启用从库复制服务，该dbpath的数据库是主库的快照，可用于快速启用同步
--autoresync    # 如果从库与主库同步数据差得多，自动重新同步;当发现从服务器的数据不是最新时，开始从主服务器请求同步数据
--oplogSize arg # 主节点的oplog日志大小，单位为M，建议设大点(更改oplog大小时，只需停主库，删除local.*，然后加--oplogSize=* 重新启动即可,*代表大小)
 * 主/从参数
 
--------------------------------------------------------------------------------
 
 
--master    # 主库模式
--slave # 从库模式
--source arg    # 从库 端口号
--only arg  # 指定单一的数据库复制
--slavedelay arg    # 设置从库同步主库的延迟时间
 * Replica set(副本集)选项：
 
--------------------------------------------------------------------------------
 
 
--replSet arg   # 设置副本集名称
 * Sharding(分片)选项
 
--------------------------------------------------------------------------------

--configsvr # 声明这是一个集群的config服务,默认端口27019，默认目录/data/configdb
--shardsvr  # 声明这是一个集群的分片,默认端口27018
--noMoveParanoia    # 关闭偏执为moveChunk数据保存
```

参考：[mongo主从配置](https://www.cnblogs.com/amoyzhu/p/7943018.html)

### 主机中存储


db.createCollection("test");
db.getCollection('test'). insert({"name":"master"})



### 验证 在slave中查看是否有此数据
./mongo host:port/db -u user -p pwd
#mongo 127.0.0.1:9430/admin -u root -p FpTH2a
db.getCollection('test').find({});
默认的SECONDARY是不允许读写的，使用命令解决
rs.slaveOk();


加Auth=true的验证

### 疑问：如果主机宕机之后，备机怎么处理？服务的可用性怎么保证？

参考：
- [mongodb 主从 部署 切换](https://blog.csdn.net/weixin_41988331/article/details/83116873)
- [mongodb主从配置及备份](https://blog.csdn.net/lelemom/article/details/82704980)