---
title: Linux-基础命令总结之systemctl
date: 2020-09-23 14:47:30
tags:
categories: Linux 
description: "服务命令  centos7 systemctl centos6以下  service "
---



systemctl也会去/etc/init.d目录下，查看，执行相关程序

- systemctl status firewalld
- systemctl list-unit-files 所有的资源
- systemctl list-units  --type=service 正在运行的   type 类型

unit 为资源

- Service unit：系统服务
- Target unit：多个 Unit 构成的一个组
- Device Unit：硬件设备
- Mount Unit：文件系统的挂载点
- Automount Unit：自动挂载点
- Path Unit：文件或路径
- Scope Unit：不是由 Systemd 启动的外部进程
- Slice Unit：进程组
- Snapshot Unit：Systemd 快照，可以切回某个快照
- Socket Unit：进程间通信的 socket
- Swap Unit：swap 文件
- Timer Unit：定时器

最常用的就是service unit

文件位置

Systemd 默认从目录/etc/systemd/system/读取配置文件。但是，里面存放的大部分文件都是符号链接，指向目录/usr/lib/systemd/system/，真正的
配置文件存放在那个目录



启动一个服务：systemctl start firewalld.service
关闭一个服务：systemctl stop firewalld.service
重启一个服务：systemctl restart firewalld.service
显示一个服务的状态：systemctl status firewalld.service
在开机时启用一个服务：systemctl enable firewalld.service
在开机时禁用一个服务：systemctl disable firewalld.service
查看服务是否开机启动：systemctlis-enabled firewalld.service
查看已启动的服务列表：systemctllist-unit-files|grep enabled
查看启动失败的服务列表：systemctl--failed

启动服务（等同于service httpd start）
systemctl start httpd.service
停止服务（等同于service httpd stop）
systemctl stop httpd.service
重启服务（等同于service httpd restart）
systemctl restart httpd.service
查看服务是否运行（等同于service httpd status）
systemctl status httpd.service
开机自启动服务（等同于chkconfig httpd on）
systemctl enable httpd.service
开机时禁用服务（等同于chkconfig httpd on）
systemctl disable httpd.service
查看服务是否开机启动 （等同于chkconfig –list）

`journalctl` 日志

Systemd 统一管理所有 Unit 的启动日志。带来的好处就是，可以只用journalctl一个命令，查看所有日志（内核日志和应用日志）。日志的配置文件是/etc/systemd/journald.conf。


[阮一峰](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)