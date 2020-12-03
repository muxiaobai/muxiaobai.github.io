---
title: Linux 基础命令总结 基本操作 文件 目录
date: 2016-06-27 20:47:05
tags: [基础命令,file,directory,tar]
categories: Linux
description: "文件、文件内容、目录、压缩打包、帮助"
---


## 目录(文件夹directory)/文件(file)相关

`ls`: 列出所有内容(list) -l 详细信息 -h 文件大小

`touch`: 新建(实际上是触摸一下,会更改文件时间戳)

`cp`: 复制文件(Copy) 

`mv`: 重命名文件(Move,移动到当前文件夹下，即重命名)

`rm`: 删除文件(Remove) -rf 强制递归删除

`ln`: 创建文件链接(link)  ln -s yuan   mubiao

##### -r: 递归

##### -f: 强制(不询问覆盖或删除)
<!--more-->
`cd`:  切换目录(Change directory)   cd -   切换到上一个目录

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

`tail`: 显示文件尾10行(-N/-nN 参数N为要显示的行号) tail -fn 300 /th/log.log  实时查看log文件

`nl`: `cat -n`的加强版(Number of Lines)  展示行号

`strings/od/xxd/`: 二进制文件

`acroread/gv/`: pdf文件和PostScript文件(Adobe Reader/Adobe PostScript/Ghostview)(大部分不支持)

`xdvi`: (TeX文本处理器输出二进制文件)(大部分不支持)

`>、<、>>` ：echo ssss > aaa.txt      写入   echo  ssss >> aaa.txt 追加ssss到文件aaa.txt   cat  < aaa.txt 读出文件到cat命令中作为输入

##### -n/N:行

##### -h/H:帮助

#### 文件属性相关

`stat`: 文件属性(status),修改时间，创建时间

`file`: 文件类型，txt ，tar.gz等，文件大小用 ls -lh

`du`: 文件占用的磁盘空间(disk usage,磁盘使用情况)

`chmod` `chown` `chgrp`: 改变权限(Change mode/owner/group)

##### -R 递归

`chattr`: 改变文件属性(Change file attributes)

`lsattr`: 显示`chattr`改变的属性(List file attributes)

#### 文件的文本性操作

`sort`: 排序

用法：sort [选项]... [文件]...
串联排序所有指定文件并将结果写到标准输出。

排序选项：

-b, --ignore-leading-blanks 忽略前导的空白区域
-d, --dictionary-order 只考虑空白区域和字母字符
-f, --ignore-case 忽略字母大小写
-g, --general-numeric-sort 按照常规数值排序
-i, --ignore-nonprinting 只排序可打印字符
-n, --numeric-sort 根据字符串数值比较
-r, --reverse 逆序输出排序结果

其他选项：

-c, --check, --check=diagnose-first 检查输入是否已排序，若已有序则不进行操作
-k, --key=位置1[,位置2] 在位置1 开始一个key，在位置2 终止(默认为行尾)
-m, --merge 合并已排序的文件，不再进行排序
-o, --output=文件 将结果写入到文件而非标准输出
-t, --field-separator=分隔符 使用指定的分隔符代替非空格到空格的转换
-u, --unique 配合-c，严格校验排序；不配合-c，则只输出一次排序结果

sort -n -r -k 2  第二列

sort -rn  数字倒序排序

sort -hr 根据K、M、G排序

`wc`: 统计(Word Count)(行  词  字节) -l line -w word -m chars

`grep` `egrep` `fgrep`: 匹配正则/扩展正则/文本

`cut`: 提取特定列

`paste`: 合并

`uniq`: 查找重复文本(unique)

`tee`:  (T形水管接口)

`tr`: 替换(Traslate)

`printf`：



## 归档(打包、压缩)相关(一般情况下,用`tar`)☆

`tar`: 打包(tape Archive)`tar -cf`代替(.tar)

__________________
##### -c 创建归档文件(Create)

##### -t 列出文件(list)

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
- tar -xvf file.tar //解压 tar包
- tar -xzvf file.tar.gz //解压tar.gz
- tar -xjvf file.tar.bz2   //解压 tar.bz2
- tar -xZvf file.tar.Z   //解压tar.Z
- unrar e file.rar //解压rar
- unzip file.zip //解压zip
- unzip -o  file.zip  覆盖解压

`zip`: Windows Zip格式压缩(.zip)(.tar.zip)

`gzip`: GNU Zip格式压缩(.gz)(.tar.gz),`tar -czf`代替

`bzip2`: BZip格式压缩(.bz2)(.tar.bz2),`tar -cjf`代替

`compress`: Unix传统格式压缩(.Z)(.tar.Z),`tar -cZf`代替

`unzip`: -o  file.zip  覆盖解压

`gunzip`: `tar -xzf`代替

`bunzip2`: `tar -xjf`代替

`uncompress`: `tar -xZf`代替

`zcat`: (支持gzip Unix传统格式)

`bzcat`: (支持BZip格式)

`metamail`: (MIME格式)

`rpm2cpio`:rpm包中抽取文件

-C 临时切换目录
tar -czvf xxx.tar.gz -C /usr/local/tomcat/web-apps ROOT/

>注意:打包和压缩是两个步骤。


#### 查找文件

`find`: 在当前目录树中查找文件

 find ./ -mtime +10 -name "*.log" | xargs ls -lh | sort -hr | head 10 查找当前文件夹下前十个最大的以log结尾最近20天的文件

`whereis`: 查找文件或者命令的源文件在哪里

`type`: 查找shell命令的类型(bash shell内部命令)

`which`: 查找shell命令所在的位置

`locate`: 创建索引 搜索/var/lib/mlocate/mlocate.db

#### 修改内容

`vi/vim`: Vi(Vim)(其他编辑器Emacs,soffice,abiword,gnumeric)

hjkl 左下上右
i ESC :  !qw quit write  ！强制

[参考Linux基础命令总结3]()

`vimtutor`: Vim帮助

## 帮助

`man`: (manual) 详细描述内容

`help`:

`info`: 

cmd --help 命令参数 --help 一般有简单描述介绍使用

