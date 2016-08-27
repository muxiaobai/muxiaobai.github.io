---
title: Sublime 下运行Java文件
date: 2016-07-19 14:57:09
tags: [Sublime,编辑器]
categories: Sublime
description: "添加批处理 Ctrl + B 运行"
---

## Sublime下运行java文件
<!--more-->
#### 首先确保命令行可以正常使用javac和java，请自行百度。

#### 在JDK的bin目录下新建runJava.bat文件，右键选编辑，复制粘贴如下代码并保存：

```
@echo off
cd %~dp1
echo Compiling %~nx1......
if EXIST %~n1.class (
del %~n1.class
)
javac -encoding UTF-8 %~nx1
if exist %~n1.class (
echo ------OUTPUT------
java %~n1
)
```
#### 打开Sublime Text 3，依次点击Preference, Browse Packages，在打开的窗口中双击User文件夹，新建文件JavaC.sublime-build，用记事本打开，粘贴下面的代码并保存关闭：
```
{
"shell_cmd": "runJava.bat \"$file\"",
"file_regex": "^(...*?):([0-9]*):?([0-9]*)",
"selector": "source.java",
"encoding": "UTF-8"
}
```
目的是
>将第一行的“shell_cmd": javac \"$file\""改成"shell_cmd": "runJava.bat \"$file\""


或者通过Package Resource Viewer来找到要修改的文件

#### 安装Package Resource Viewer。同上面的步骤，打开package control，输入PackageResourceViewer:Open Resource，回车后输入java，回车后输入Javac.sublime-build,就能够打开我们所需要修改的文件

>主要是用批处理文件处理sublime中的文件，用runJava.bat 代替javac和java

## 保存后即可在Sublime Text 3中按 **`Ctrl+B`** 编译Java运行文件，这种方法的缺点是无法在控制台输入，如果程序需要输入内容，则直接报错