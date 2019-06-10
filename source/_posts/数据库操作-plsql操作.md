---
title: 数据库操作-plsql操作
date: 2018-11-07 05:58:38
tags: [常用代码]
categories: [SQL,数据库]
description: "plsql的一些操作"
---


PLSQL的一些操作，
<!--more-->

####  快捷键

在Tools->Perferences->Editor中Autoreplaces选择配置的shortcuts文件

C:\ProgramTool\PLSQL Developer\short.txt

```
sf = select * from 
w = where 
sf = select t.*, t.rowid from  
sc = select count(1) from 
df = delete from 

```

#### Session查看 

plsql 工具   Tools ----->Sessions---------> 查看

#### 数据库比对工具

plsql Tools ------->Compare User Objects 选择另外一个库

#### 导入csv数据

需要将csv另存为csv

plsql Tools ------->Text Importer---->Open data file  --->Data to Oracle 选择Owener Table

#### 记住密码多个账号，账号密码等

Tools->Preferences

tools -> Preferences -> User Interface - Options
勾选 Autosave username, ，保存即可

tools -> Preferences -> Oracle -> logon History

Definition->Store history,Store with password

->fixed user
按格式：user/password@数据库 添加一个fixed user保存即可
zzz/sdsdfs@192.168.1.12:1555/orcl


[参考](https://www.cnblogs.com/Chary/p/No00008F.html)