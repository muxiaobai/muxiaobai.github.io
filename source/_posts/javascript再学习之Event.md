---
title: javascript再学习之Event
date: 2017-08-09 12:03:00
tags: 前端
categories: [javascript]
description: "顺着上一篇DOM结束，本来是CSS，但自己对事件了解的很少，就先看Event了"
---


## Event


addEventListener/attachEvent

removeEventListener/detachEvent

通过addEventListener()添加的事件处理程序只能使用removeEventListener()来移除；
移除时传入的参数与添加处理程序时使用的参数相同。这也意味着通过addEventListener()添加的匿名函数无法移除
布尔值参数是true，表示在捕获阶段调用事件处理程序；如果是false，表示在冒泡阶段调用事件处理程序。

```
document.body.addEventListener('touchmove', function (event) {
    event.preventDefault();
},false);
document.body.removeEventListener('touchmove', function (event) {
    event.preventDefault();
},false);
这样是不能移除的，因为第二个函数是一个新的空间函数，既是写的样子和第一个一样。

```
一：相同事件绑定和解除，需要使用共用函数；

二：共用函数不能带参数；

### 捕获与冒泡

Event原理


一、事件捕获阶段

二、事件目标阶段

三、事件起泡阶段

jQuery

在绑定的时候做了包装处理
在执行的时候有过滤器处理

[ JS添加事件和解绑事件：addEventListener()与removeEventListener()](http://blog.csdn.net/bingkingboy/article/details/50160221)
[addEventListener()、attachEvent()和removeEventListener()、detachEvent()的区别？](http://blog.csdn.net/itpinpai/article/details/50915771)