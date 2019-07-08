---
title: tools-ide快捷操作
date: 2019-06-27 19:28:57
tags: ['插件','工具']
categories: 工具
description: "日常工具快捷操作"
---


## sublime

sublime 工具插件
package control
ConvertToUTF8


激活sublime

C:\Windows\System32\drivers\etc  host


127.0.0.1 www.sublimetext.com
127.0.0.1 license.sublimehq.com
127.0.0.1 45.55.255.55
127.0.0.1 45.55.41.223


----- BEGIN LICENSE -----
sgbteam
Single User License
EA7E-1153259
8891CBB9 F1513E4F 1A3405C1 A865D53F
115F202E 7B91AB2D 0D2A40ED 352B269B
76E84F0B CD69BFC7 59F2DFEF E267328F
215652A3 E88F9D8F 4C38E3BA 5B2DAAE4
969624E7 DC9CD4D5 717FB40C 1B9738CF
20B3C4F1 E917B5B3 87C38D9C ACCE7DD8
5F7EF854 86B9743C FADC04AA FB0DA5C0
F913BE58 42FEA319 F954EFDD AE881E0B
------ END LICENSE ------

ZYNGA INC.
50 User License
EA7E-811825
927BA117 84C9300F 4A0CCBC4 34A56B44
985E4562 59F2B63B CCCFF92F 0E646B83
0FD6487D 1507AE29 9CC4F9F5 0A6F32E3
0343D868 C18E2CD5 27641A71 25475648
309705B3 E468DDC4 1B766A18 7952D28C
E627DDBA 960A2153 69A2D98A C87C0607
45DC6049 8C04EC29 D18DFA40 442C680B
1342224D 44D90641 33A3B9F2 46AADB8F


## IDEA   


### 插件
Key promoter X  展示快捷键
Alibaba Java Coding Guidelines
SonarLint  代码检查

### 注释

/**
* Project Name:${project_name}
* File Name:${file_name}
* Package Name:${package_name}
* Date:${date} ${time}
* Copyright (c) ${year}, All Rights Reserved.
*
*/

${filecomment}

${package_declaration}
/**
* ClassName:${type_name} 
* Function: ${todo} 
* Reason: ${todo} 
* Date: ${date} ${time} 
* @author Mu Xiaobai
* @version 
* @since JDK 1.8 
*/
${typecomment}
${type_declaration}

aa 包
/**
* Project Name:$project_name$
* File Name:$file_name$
* Package Name:$package_name$
* Date:$date$ $time$
* Copyright (c) $year$, All Rights Reserved.
*
*/
ss 类
/**
* @ClassName: $class_name$ 
* @Function: //TODO 
* @Reason: //TODO
* @Date: $date$ $time$ 
* @author Mu Xiaobai
* @version 
* @since JDK 1.8 
*/
zz 方法
/* 
* @name: $enclosing_method$
* @description: TODO 
* @param $param$
* @return: $return$ 
* @date: $date$ $time$
* @auther: $user$
* 
*/

### 设置


### 快捷键
ctrl + shift + r 替换
ctrl + shift + f 查找

alt + ←/→  鼠标上次的位置/下次的位置
ctrl + alt + 鼠标左键 查看方法的实现类


ctrl + shift + u 大小写替换


ctrl + x 剪切
ctrl + d 复制粘贴
ctrl + / 单行注释
ctrl + shift + / 选中内容多行注释


ctrl + z 撤回
ctrl + shift + z 撤回的撤回

ctrl +alt + s 设置
ctrl +alt +shift + s 本项目设置 打包

参考[IntelliJ-IDEA-Tutorial](https://github.com/judasn/IntelliJ-IDEA-Tutorial/blob/master/keymap-introduce.md)
## navicat

ctrl + shift + r  选中内容执行
ctrl + r 执行当期窗口所有
ctrl + q 打开新执行窗口
ctrl + w 关闭当期窗口


## plsql

F8 执行选中

设置AutoReplace
```
sf = select t.* ,t.rowid from 
s = select * from 

```
打开PL/SQL，在Tools->Perferences->Editor中Autoreplace，勾选Enabled，选择配置的AutoReplace.txt文件，点击OK即可。


## eclipse

### 快捷键
ctrl+shift+G查看方法被调用
ctrl+alt+R 重启或者启动tomcat
ctrl+shift+T 查找文件在哪里 匹配java
ctrl+shift+R resource查找所有资源
ctrl+h file search Containing text 搜索包含某个字符串的所有文件
ctrl+shift+c注释/反注释
ctrl + alt + H查看方法被调用

ctrl+shift+L  查看所有的快捷键
  
ctrl+alt+G  搜索文本

### 插件

- svn - http://subclipse.tigris.org/update_1.8.x
- jd-core - http://jd.benow.ca/jd-eclipse/update
- zookeeper：  http://www.massedynamic.org/eclipse/updates/
- Enhanced Class Decompiler  http://feeling.sourceforge.net/update

反编译

重启之后，在窗口菜单栏点击Widow->Preference->General->Editors->File Associations,将FileType里的*.class和*.class without source的Associated editors下面的Class File Editor设置成default即可
