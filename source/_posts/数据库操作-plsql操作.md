---
title: 数据库操作-plsql操作
date: 2018-11-07 05:58:38
tags: [常用代码]
categories: [SQL]
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