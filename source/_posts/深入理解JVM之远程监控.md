---
title: 深入理解JVM之远程监控
date: 2018-11-22 07:21:35
tags: [jvm,tomcat]
categories: java
description: "JVM配置远程监控,配置"
---

远程监控jconsole和jvisualvm
<!--more-->

修改服务器tomcat的bin/catalina.bat文件,

新建setenv.sh,给权限

```
JAVA_OPTS="$JAVA_OPTS -Djava.rmi.server.hostname=192.168.202.121 -Dcom.sun.management.jmxremote"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.port=12345"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.authenticate=true"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.pwd.file=$JAVA_HOME/jre/lib/management/jmxremote.password"

```


配置权限文件

```

[root@localhost bin]# cd $JAVA_HOME/jre/lib/management

[root@localhost management]# cp jmxremote.password.template jmxremote.password

[root@localhost management]# vi jmxremote.password

monitorRole QED
controlRole chenqimiao

[root@localhost management]# chmod 600 jmxremote.password jmxremote.access

```

linux:  netstat -ano | grep 12345
win :   netstat -ano | findstr 12345


修改jvisualvm工具无法下载插件问题。
修改插件>>>设置中的地址
在[visual VM 网址查询](https://visualvm.github.io/pluginscenters.html)

- [JConsole远程连接](https://www.cnblogs.com/think-in-java/p/6138439.html)
- [监控JVM带main方法](https://blog.csdn.net/yunzhu666/article/details/8662045)