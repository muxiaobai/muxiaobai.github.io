---
title: Linux 基础命令总结3
date: 2017-09-17 00:43:52
tags: 基础命令
categories: Linux
description: "软件安装 yum，Vim"
---

## 安装

#### yum 


`yum list` 

`yum install`

`rpm`

-a：查询所有套件； 
-b<完成阶段><套件档>+或-t <完成阶段><套件档>+：设置包装套件的完成阶段，并指定套件档的文件名称； 
-c：只列出组态配置文件，本参数需配合"-l"参数使用； 
-d：只列出文本文件，本参数需配合"-l"参数使用； 
-e<套件档>或--erase<套件档>：删除指定的套件； 
-f<文件>+：查询拥有指定文件的套件； 
-h或--hash：套件安装时列出标记； 
-i：显示套件的相关信息； -i<套件档>或--install<套件档>：安装指定的套件档； 
-l：显示套件的文件列表； -p<套件档>+：查询指定的RPM套件档； 
-q：使用询问模式，当遇到任何问题时，rpm指令会先询问用户； 
-R：显示套件的关联性信息； 
-s：显示文件状态，本参数需配合"-l"参数使用； 
-U<套件档>或--upgrade<套件档>：升级指定的套件档； 
-v：显示指令执行过程； 
-vv：详细显示指令执行过程，便于排错。


`yum`

install：安装rpm软件包； 
update：更新rpm软件包； 
check-update：检查是否有可用的更新rpm软件包； 
remove：删除指定的rpm软件包； 
list：显示软件包的信息； 
search：检查软件包的信息； 
info：显示指定的rpm软件包的描述信息和概要信息； 
clean：清理yum过期的缓存； 
shell：进入yum的shell提示符； 
resolvedep：显示rpm软件包的依赖关系； 
localinstall：安装本地的rpm软件包； 
localupdate：显示本地rpm软件包进行更新；
deplist：显示rpm软件包的所有依赖关系。

## Vim

- 选中 v，移动光标 然后y复制  
- 复制yy 3yy 复制光标行，复制光标及下面两行
- 粘贴p 光标处粘贴复制内容
- 删除dd 3dd 删除光标行，删除光标及以下三行
- u 撤销  Ctrl +R 回撤
- Ctrl f 下一页 Ctrl b 上一页
- 行头行尾 0/^行首 $ 行尾
- 文件头文件尾 gg/1G 文件头部  3G 第三行 G 文件尾部 
- /查找匹配

![图解命令](Linux-基础命令总结3/1353759337_6781.png)
![图解命令](Linux-基础命令总结3/160604162658756.gif)


## 高级

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

[CMD快捷键](https://www.cnblogs.com/webzhangnan/p/3221410.html)