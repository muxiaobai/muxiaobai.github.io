---
title: Linux 基础命令总结2 用户 权限 服务
date: 2016-07-07 13:02:34
tags: [基础命令,系统]
categories: Linux
description: "用户，网络(端口占用等)、服务(自启动)、系统环境、权限、进程、任务调度、工作"
---

## 打印、输出

#### 打印

`lpr`  `lpq` `lprm`

#### 输出

`echo`  `printf` `yes` `seq` `clear`
<!--more-->
## 用户、主机环境 

#### 用户

`logname`: 登录名

`whoami`: 当前有效的用户名

`id`: pid和gid

`who`: 详细列出所有登录用户，看谁正在使用，展示IP

`users`: 简单列出所有登录用户

`finger`: 打印用户信息

`last`: 确定之前谁在什么时候登录这台主机，历史登录用户

#### 主机

`printenv`:打印环境变量

`uname`: 系统内核信息（Unix name）

##### -a 所有 

##### –r内核版本（release） 

##### -s内核名称

`hostname`:系统主机名

----

-s, --short           short host name

-a, --alias           alias names

-i, --ip-address      addresses for the hostname  

-I, --all-ip-addresses all addresses for the host

-f, --fqdn, --long    long host name (FQDN)

-A, --all-fqdns        all long host names (FQDNs)

-d, --domain          DNS domain name `dominname` 

-y, --yp, --nis       NIS/YP domainname   `dnsdominname` `ypdominname` `nisdominname`

-F, --file            read hostname or NIS domainname from given file   

-----

`history` 历史命令

## 权限(目录和文件)和用户，组


#### rwx权限 读写执行

 >所有者 所属者 其他人

默认：`umask` 

改变权限：`chmod` `chown` `chgrp`

#### 高级权限管理ACL SODO 文件特殊权限

##### ACL权限(解决用户身份不够的情况)(目录和文件)
 
 >首先要看ACL权限是否开启 `dumpe2fs -h /dev/xvdal`
 
 >`setfacl`
 
 >`getfacl` 文件或目录后面有一个[+]号
 
 >mask 用来控制最大权限

##### sudo(超级用户root执行的系统命令)(命令)

 >visudo(修改/etc/sudoers.tmp文件)
 > sudo -l 查看被添加的权限

##### SetUID SetGID Sticky BIT(文件特殊权限)(一般不使用)

 >SetUID 可执行文件有执行权限的时候才有SUID  USER[s=S+x]S无效
 
 >`chmod 4755[u+s] 文件名`
 
 >`chmod 0755[u-s] 文件名`
 
 >SetGID 可执行文件有执行权限的时候才有SGID,目录的rx权限  GROUP组身份是我
 
 >`chmod 2755[g+s] 文件名`
 
 >`chmod 0755[u-s] 文件名`
 
 >Sticky BIT  SBIT粘着位作用,仅针对目录，其他人有rwx权限 
 
 >`chmod 1755[o+t] 目录`
 
 >`chmod 0755[o-t] 目录`
 
 > i[insert插入]/a[append追加] 不可改变位权限
 
 >`chattr [a/i]`

 >`lsattr`

`pam` `selinux`

#### 用户：

`useradd` `userdel` `usermod`

#### 所属组：

`groups` `groupadd` `groupdel` `groupmod`


useradd 用户名 -g 组名–G 组名-d Home 目录名-p 密码

useradd usenmae –g usenmae  –p ora123 

passwd  username
/etc/passwd
cat /etc/group  查看所有所属组

change owner  change modified

chmod -R g+w  group增加 write权限

chown -R owner:group /home/newname  把/home/newname的所有者改为owner，所属组改为group

-R 递归

读r=4  写w=2  执行x=1  用户u，组g，其他用户o

常见 文件644 -rx-r--r--   drwxr-xr-x 文件夹755

- u 代表所有者（user）
- g 代表所有者所在的组群（group）
- o 代表其他人，但不是u和g （other）
- a 代表全部的人，也就是包括u，g和o
- r 表示文件可以被读（read）
- w 表示文件可以被写（write）
- x 表示文件可以被执行（如果它是程序的话）



`chmod go-rw xxx.xxx`

## 网络和连接登录，上传下载文件

`arp`: 硬件地址(IP ----> MAC地址)

`traceroute`: 到某一个地址的路由信息

`route`: 路由表

`ifconfig`: 配置网络信息

`ping`: 检测网络畅通

`host`: 检测分析域名是否正常

`netstat`: 查看网络状态、端口状态 -tulp   -tnaop  ☆

-a 端口
-t tcp
-u udp
-l listener
-p program name 
      
      netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

解析：
- CLOSED  //无连接是活动的或正在进行
- LISTEN  //服务器在等待进入呼叫
- SYN_RECV  //一个连接请求已经到达，等待确认
- SYN_SENT  //应用已经开始，打开一个连接
- ESTABLISHED  //正常数据传输状态/当前并发连接数
- FIN_WAIT1  //应用说它已经完成
- FIN_WAIT2  //另一边已同意释放
- ITMED_WAIT  //等待所有分组死掉
- CLOSING  //两边同时尝试关闭
- TIME_WAIT  //另一边已初始化一个释放
- LAST_ACK  //等待所有分组死掉

`ip`: iproute2中的命令(以上所有命令基本上都可以用这个命令来使用)

`net`: 无

`msg`:禁用

`nbtstat`:  无

`wget`: 下载资源

`curl` : 

[url]

-X POST GET  方法
-H "Content-Type:application/json" 请求头Header
-F "filename=@/home/test/file.tar.gz;type=application/octet-stream"  file文件上传
-d "action=del&name=archer" form data
-u 用户
-v 查看请求

curl -XPOST http://ip:port -H "Content-Type=multipart/form-data"  -F "file=@app-search-component.zip"

curl -u elastic:123123 http://ip:port

curl -H 'Content-Type: application/json' -XPOST  -u my_admin::my_pwd 'http://localhost:8080/' -d '{"password" : "123123"}'

curl -u 'u:p' -H 'Content-type:application/json' -XPOST -d '@xxx.json' http://ip:port/path > result.txt

远程连接：`ssh` `scp` `sftp` `telnet`(一般禁用) `ftp`(不常用)

`sz` file 下载文件到本机

`rz` 上传

`scp /opt/local root@192.168.1.2:/opt/remote` 文件到192.168.1.2服务器上

`scp  root@192.168.1.2:/opt/remote /opt/local` 文件从192.168.1.2服务器上拉取


安装rzsz

wget http://www.ohse.de/uwe/releases/lrzsz-0.12.20.tar.gz
tar zxvf lrzsz-0.12.20.tar.gz && cd lrzsz-0.12.20
./configure --prefix=/usr/local/'name' && make && make install 
上面安装过程默认把lsz和lrz安装到了/usr/local/bin/目录下，现在我们并不能直接使用，下面创建软链接，并命名为rz/sz：
cd /usr/bin

ln -s /usr/local/bin/lrz rz

ln -s /usr/local/bin/lsz sz

或者 
yum install -y lrzsz

#### 端口占用6中方法

ss -tnlp  
netstat
lsof
fuser 
nmap (NetWork Mapper) 网络监测和安全审计工具，可能无此命令
systemctl systemd系统的控制管理器和服务管理器 可能无此命令


netstat -tlnaop

-c 字符串  -u 用户名  -p pid


lsof  进程打开或使用、调用的文件信息☆

lsof -i:端口号 用于查看某一端口的占用情况，比如查看8000端口使用情况，lsof -i:8000

## 进程、系统资源(磁盘)

#### 进程

`ps`: 查看进程

##### aux BSD

##### -el Linux
-----
ps命令常用用法（方便查看系统进程）

- 1）ps a 显示现行终端机下的所有程序，包括其他用户的程序。
- 2）ps -A 显示所有进程。
- 3）ps -c列出程序时，显示每个程序真正的指令名称，而不包含路径，参数或常驻服务的标示。
- 4）ps -e 此参数的效果和指定"A"参数相同。
- 5）ps e 列出程序时，显示每个程序所使用的环境变量。
- 6）ps f 用ASCII字符显示树状结构，表达程序间的相互关系。
- 7）ps -H 显示树状结构，表示程序间的相互关系。
- 8）ps -N 显示所有的程序，除了执行ps指令终端机下的程序之外。
- 9）ps s 采用程序信号的格式显示程序状况。
- 10）ps S 列出程序时，包括已中断的子程序资料。
- 11）ps -t<终端机编号> 　指定终端机编号，并列出属于该终端机的程序的状况。
- 12）ps u 　以用户为主的格式来显示程序状况。
- 13）ps x 　显示所有程序，不以终端机来区分。

最常用的方法是ps -aux,然后再利用一个管道符号导向到grep去查找特定的进程,然后再对特定的进程进行操作。

ps -ef | grep tomcat 查看tomcat进程☆


##### 检查tomcat/nginx 并发数，连接数等

内部的应用级别的：
server {
    listen  *:80 default_server;
    server_name *.jiloc.com jiloc.com;
    location /ngx_status   {
        stub_status on;
        access_log off;
        #allow 127.0.0.1;
        #deny all;
    }
}

curl http://127.0.0.1/ngx_status

浏览器 http://127.0.0.1/jStatus



外部的，命令级别的：

可查看所有建立连接的详细记录: netstat -nat | grep ESTABLISHED|wc  

netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

ps -ef|grep tomcat

查看tomcat的线程数: ps -Lf pid|wc -l
查看tomcat的并发数: netstat -an|grep pid |awk '{count[$6]++} END{for (i in count) print(i,count[i])}'


[Linux 下Web服务器 Nginx 状态监控 查看nginx当前并发 连接 请求状态](https://www.jiloc.com/42193.html)
[linux进程、线程状态 tomcat线程数 并发数查看](https://blog.csdn.net/wngua/article/details/70904991)
[inux查看连接数，并发数](http://duanfei.iteye.com/blog/1894387)

---------

##### 输出信息

--------
USER 该进程由哪个用户产生的

PID 该进程ID号

%CPU  该进程占用CPU资源的百分比

%MEM  该进程占用物理内存资源的百分比

VSZ 该进程占用虚拟内存大小，单位KB

RSS 该进程占用实际物理内存的大小，单位KB

TTY ？内核调用，该进程在哪一个终端上运行的，tty1-tty7代表本地控制台终端，tty1-tty6本地字符界面，tty7图形界面 pts/0-255虚拟终端（远程登录，远程终端）

STAT  进程状态  R：运行 S：睡眠 T：停止s：包含子进程+：位于后台

START 该进程的启动时间 

TIME   该进程占用CPU的运算时间

COMMAND　产生该进程的命令名

--------------

`pstree` :树形进程展示 

##### -p 显示进程PID（详情）

##### -u 显示进程所属用户(USER)

`top`: 系统健康状况

默认每三秒更新一次 默认CPU占用率排序

当前时间|运行时间|用户|平均负载load average： 1min 5min 15min
总数| 状态（运行、睡眠、停止、僵尸：正在停止但是没有完全停止）
CPU | 用户us| 系统sy|  改变过优先级的用户ni|空闲率id|  等待输入/输出wa|硬中断hi |软中断si| 虚拟时间st（steal time）

|MEM（物理内存KB） |总total |空闲free| 使用used | 缓冲buff/cache  
|SWAP（交换分区KB）|总total |空闲free| 使用used | 

-d  每隔几秒更新一次

-c 显示全命令

?h：交互模式帮助

P：CPU使用率排序（默认）

M：内存使用率排序

N：PID排序

q：退出

`uptime`: 显示启动时间和平均负载(top命令的第一行)(`w`也可以看到此信息,还有用户信息)()

`kill`: 杀死进程(-l参数:查看信号) 

##### kill –l进程ID

kill [-1重启] [-9强制] 进程ID

`killall`: 进程名  

`pkill`: 进程名

pkill  -t 终端号（TTY） 按终端号剔除用户（w显示用户）

`nice`:优先级


`renice`:优先级

#### CPU

总核数 = 物理CPU个数 X 每颗物理CPU的核数 

总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

查看每个物理CPU中core的个数(即核数)

cat /proc/cpuinfo| grep "cpu cores"| uniq

查看逻辑CPU的个数

cat /proc/cpuinfo| grep "processor"| wc -l

查看CPU信息（型号）
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c

#### 磁盘

挂载卸载：`mount` `umount` 


磁盘空间：
`du` estimates and displays the disk space used by files    du  -h

`df`  df -sh 查看存储大小

`dmesg`

##### dmesg 开机内核监测信息 | grep CPU

`free`  free -m 

##### free 查看内存-b –k –m –g

`parted` `sfdisk` `fdisk` 

`mkfs` 

`floppy` 

## 工作

`jobs`: 列出所有正在运行的任务

`fg`: 恢复任务，前台运行该任务(foreground)

`bg`: 将任务放到后台运行(background)

`Ctrl+Z`: 暂停当前任务,和`bg`类似

`命令后加 &`: 将任务放到后台运行

`suspend`: 暂停shell

## 定时任务、任务控制   

`crond`: 定时任务

crond后台守护d  cron

service crond restart重启服务

chkconfig crond on检查是否启动

`crontab`: 循环定时任务

-----

##### -e 编辑定时任务

##### -l 查看任务

##### -r 删除当前用户的所有任务

----

\*  \*  \*  \*  \*  command

一个小时当中的第几分钟0-59

一天当中的第几个小时0-23

一个月当中的第几天1-31

一年中的的第几个月1-12

一周当中的第几天0-7（0、7代表星期天）

\* 任意时间

, 一个不连续的时间

\- 连续的时间范围

*/n 每隔n执行

`sleep`: ()

`watch`: ()

`at`: ()

## 服务


正常情况下，使用绝对路径  /etc/init.d  启动脚本位置

Red Hat `service` `ntsysv` 

默认启动项：`chkconfig` 


systemctl enable xxx-service
systemctl list-unit-files |  grep enabled

开机自启动
新建一个脚本zookeeper
为新建的脚本zookeeper添加可执行权限，命令是:chmod +x zookeeper
把zookeeper这个脚本添加到开机启动项里面，命令是： chkconfig --add zookeeper
如果想看看是否添加成功，命令是：chkconfig --list


## 安装软件

Red Hat `rpm` `yum`   rpm -ivh xxxx.rpm ; yum install

Debian `dpkg` `aptitude`


[安装软件](http://muxiaobai.github.io/2017/09/17/Linux-%E5%9F%BA%E7%A1%80%E5%91%BD%E4%BB%A4%E6%80%BB%E7%BB%933/)

##附件

`cal`: 日历(Calendar)

`date`: 日期 修改时间并同步到硬件上 date -s "2018-12-03 16:10:10" & hwclock --systohc

`dc`:计算器

`expr`:计算器


## 登录、注销和关机

`shutdown`: 

##### -r 重启

##### -h 关机

`logout`

`exit`

