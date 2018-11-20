---
title: javascript-常用代码
date: 2018-10-09 06:31:08
tags: 前端
categories: javascript
description: ""
---

#### select option选择

```
var options=$("#select option:selected"); //获取选中的项
alert(options.val()); //拿到选中项的值
alert(options.text()); //拿到选中项的文本
alert(options.attr('url')); //拿到选中项的url值

```
#### 数组对象去重


```

var arr=[{id:1,name:"z"},{id:2,name:"g"},{id:1,name:"z"];
arr = unique(arr,"id");
console.log(arr);

function arrayUnique2(arr, name) {
		  var hash = {};
		  return arr.reduce(function (item, next) {
		    hash[next[name]] ? '' : hash[next[name]] = true && item.push(next);
		    return item;
		  }, []);
}

```