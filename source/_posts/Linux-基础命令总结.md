---
title: Linux 基础命令总结
date: 2016-06-27 20:47:05
tags: [基础命令,file,directory,tar]
categories: Linux
description: "文件、文件内容、目录、压缩打包、帮助"
---


## 目录(文件夹directory)/文件(file)相关

`ls`: 列出所有内容(list)

`touch`: 新建(实际上是触摸一下,会更改文件时间戳)

`cp`: 复制文件(Copy)

`mv`: 重命名文件(Move,移动到当前文件夹下，即重命名)

`rm`: 删除文件(Remove)

`ln`: 创建文件链接(link)

##### -r: 递归

##### -f: 强制(不询问覆盖或删除)
<!--more-->
`cd`:  切换目录(Change directory)

`mkdir`: 新建目录(Make directory)

`rmdir`: 删除目录(Remove directory) (`rm -r`: 强制删除非空目录)(`rm -rf`)

##### -p: 递归创建/删除非空目录

`pwd`: 显示当前工作目录的绝对路径(Print work directory)

`basename`: 显示文件名

`dirname`: 显示路径

## 文件内容相关

#### 查看内容

`cat`: 查看内容(Catenate)

`less`: 分页满屏显示 (less is more)

`more`: 和less功能相似，但是没有less强大

`head`: 显示文件前10行(-N/-nN 参数N为要显示的行号)

`tail`: 显示文件尾10行(-N/-nN 参数N为要显示的行号)

`nl`: `cat -n`的加强版(Number of Lines)

`strings/od/xxd/`: 二进制文件

`acroread/gv/`: pdf文件和PostScript文件(Adobe Reader/Adobe PostScript/Ghostview)(大部分不支持)

`xdvi`: (TeX文本处理器输出二进制文件)(大部分不支持)

##### -n/N:行

##### -h/H:帮助

#### 文件属性相关

`stat`: 文件属性(status)

`file`: 文件类型

`du`: 文件占用的磁盘空间(disk usage,磁盘使用情况)

`chmod` `chown` `chgrp`: 改变权限(Change mode/owner/group)

##### -R 递归

`chattr`: 改变文件属性(Change file attributes)

`lsattr`: 显示`chattr`改变的属性(List file attributes)

#### 文件的文本性操作

`sort`: 排序

`wc`: 统计(Word Count)(行  词  字节)

`grep` `egrep` `fgrep`: 匹配正则/扩展正则/文本

`cut`: 提取特定列

`paste`: 合并

`uniq`: 查找重复文本(unique)

`tee`:  (T形水管接口)

`tr`: 替换(Traslate)

`printf`：

#### 高级

`awk` `gawk`: 

`sed`: 

`m4`: 

`gcc`: gcc/g++程序,语言C、C++

`perl`: perl程序,语言Perl

`python`: python程序,语言Python

`java`: javac程序,语言java

`mono`: mono程序,语言.NET

`php`: php程序,语言PHP

`ruby`: ruby程序,语言Ruby

#### 归档(打包、压缩)相关(一般情况下,用`tar`)

`tar`: 打包(tape Archive)`tar -cf`代替(.tar)

__________________
##### -c 创建归档文件(Create)

##### -t 列出文件()

##### -x 解压()

##### -r 在已有的归档文件后添加新的文件

##### -u 在已有的归档文件后添加新的文件或者修改原有归档文件
__________________
##### -z (gzip)

##### -j (bzip2)

##### -Z (Unix传统格式compress)
__________________
##### -v 打印详细信息(View)

##### -f 从文件下打包(File)
__________________
`zip`: Windows Zip格式压缩(.zip)(.tar.zip)

`gzip`: GNU Zip格式压缩(.gz)(.tar.gz),`tar -czf`代替

`bzip2`: BZip格式压缩(.bz2)(.tar.bz2),`tar -cjf`代替

`compress`: Unix传统格式压缩(.Z)(.tar.Z),`tar -cZf`代替

`unzip`: 

`gunzip`: `tar -xzf`代替

`bunzip2`: `tar -xjf`代替

`uncompress`: `tar -xZf`代替

`zcat`: (支持gzip Unix传统格式)

`bzcat`: (支持BZip格式)

`metamail`: (MIME格式)

`rpm2cpio`:rpm包中抽取文件

>注意:打包和压缩是两个步骤。

#### 查找文件

`find`: 在当前目录树中查找文件

`whereis`: 查找文件或者命令的源文件在哪里

`type`: 查找shell命令的类型(bash shell内部命令)

`which`: 查找shell命令所在的位置

`locate`: 创建索引 搜索/var/lib/mlocate/mlocate.db

#### 修改内容

`vi/vim`: Vi(Vim)(其他编辑器Emacs,soffice,abiword,gnumeric)

`vimtutor`: Vim帮助

## 帮助

`man`: (manual)

`help`:

`info`: 


