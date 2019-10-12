---
title: javascript-常用代码
date: 2018-10-09 06:31:08
tags: 前端
categories: javascript
description: "前端主要是JS的用法，获取参数，去重，"
---

#### select option选择

```
var options=$("#select option:selected"); //获取选中的项
alert(options.val()); //拿到选中项的值
alert(options.text()); //拿到选中项的文本
alert(options.attr('url')); //拿到选中项的url值

```

#### ajax 设置请求头

```
 $.ajaxSetup({
	beforeSend:function(request, settings) {
		if (token) {
		request.setRequestHeader("Authorization", token);
		}
	}
});

```

#### Ajax,设置时间超时 

```
var ajaxTimeoutTest = $.ajax({
　　url:'',  //请求的URL
　　timeout : 1000, //超时时间设置，单位毫秒
　　type : 'get',  //请求方式，get或post
　　data :{},  //请求所传参数，json格式
　　dataType:'json',//返回的数据格式
　　success:function(data){ //请求成功的回调函数
　　　　alert("成功");
　　},
　　complete : function(XMLHttpRequest,status){ //请求完成后最终执行参数
　　　　if(status=='timeout'){//超时,status还有success,error等值的情况
 　　　　　 ajaxTimeoutTest.abort();
　　　　　  alert("超时");
　　　　}
　　}
});

function request(url, data, success_callback,error_callback) {
    console.log("url:"+url);

var xhr = $.ajax({
        //提交数据的类型 POST GET
        type: "POST",
        //提交的网址
        url: url,
        //提交的数据
        data: data,
        //设置为同步
        async:false,
        // 设置超时的时间20s
        timeout:20000,
        //返回数据的格式
        datatype: "json", //"xml", "html", "script", "json", "jsonp", "text".
        xhrFields: {
            withCredentials: true
        },
        crossDomain: true,
        //在请求之前调用的函数
        beforeSend: function () {
 
        },
        //调用执行后调用的函数
        complete: function (XMLHttpRequest, textStatus) {
            if(textStatus == 'timeout'){//超时,status还有success,error等值的情况
                if (error_callback != null && error_callback != "") {            
                    error_callback();
                };
            }
        },
        //成功返回之后调用的函数             
        success: function (response) {
            handleResponse(response, success_callback,error_callback);
        },
      
        //调用出错执行的函数
        error: function () {
            //请求出错处理
            console.log("error");
        }
    });
}

```
#### xhr  下载数据可以，加请求头

调用方式
 $("#export").click(function(){
      // window.location.href ='http://192.0.0.1:8080/exportData?id=' + id;
          createxhr('http://192.0.0.1:8080/exportData?id=' + id);
  });
```
function createxhr(url){
    var xhr = new XMLHttpRequest();
    var formData = new FormData();
    // formData.append("ss","ss");
    //如果是post，在formData append参数
    xhr.open('get',url);  
    xhr.setRequestHeader("Authorization", sessionStorage.getItem('token'));
    xhr.responseType = 'blob';
    xhr.onload = function (e) {
        if (this.status == 200) {
          var blob = this.response;
          var filename = "filename.xlsx";
          blob.type ="application/octet-stream";
          if (window.navigator.msSaveOrOpenBlob) {
              navigator.msSaveBlob(blob, filename);
          } else {
            var a = document.createElement('a');
            var url = "";//createObjectURL(blob);
            if(window.URL){
                url = window.URL.createObjectURL(blob);
            }else{
                url = window.webkitURL.createObjectURL(blob);
            }
            a.href = url;
            a.download = filename;
            document.body.appendChild(a);
            a.click();
            window.URL.revokeObjectURL(url);
          }
        }
    };
    xhr.send(formData);
  }
```
测试chrome通过，IE11通过，

#### xhr 上传数据

可以使用FormData来进行。



####  在类数组对象中找到其中符合条件的数据

```
var data = [{menuId:1,menuName:"菜单",parentMenuId:0},
{menuId:2,menuName:"角色",parentMenuId:1},
{menuId:3,menuName:"人员",parentMenuId:1}];
getOne function(){
	var menuId = 1;
	function objFn(value, index, arr){
		return value["menuId"]== menuId;
	}
	return data[data.findIndex(objFn)];//

};
```
返回：{menuId:1,menuName:"菜单",parentMenuId:0}
ES6语法 findIndex();

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


reduce函数

```
#### 数组去重

```

//数组去重
function uniqueList(array){
    var r = [];
    for(var i = 0, l = array.length; i < l; i++) {
        for(var j = i + 1; j < l; j++)
            if (JSON.stringify(array[i]) == JSON.stringify(array[j])) j = ++i;
        r.push(array[i]);
    }
    return r;
}


```
#### 数组delete 之后length无效

```
//删除的数组obj, 删除第i个数据
//delete obj[i];
function changeLength(obj,i){
	return  obj.slice(0,i).concat(obj.slice(i+1,obj.length));
}

```
#### 深拷贝

```

普通数组直接使用[].concat(_test)
对象数组：[].concat(JSON.parse(JSON.stringify(_test)))

```

#### 替代eval方案

```
function  evil(fn)
{
    var Fn = Function;
    return new Fn('return ' + fn)(); 
}


evil("function(){console.log(1111);console.log(this)}()")

```

#### 获取地址中的参数

```
//正则
function getUrlParam(name) {
 	var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)"); //构造一个含有目标参数的正则表达式对象
    var r = window.location.search.substr(1).match(reg);  //匹配目标参数
    if (r != null) return unescape(r[2]); return null; //返回参数值
     }

//第二种，循环
function getQueryVariable(variable)
{
    var query = window.location.search.substring(1);
    var vars = query.split("&");
    for (var i=0;i<vars.length;i++) {
        var pair = vars[i].split("=");
        if(pair[0] == variable){return pair[1];}
    }
    return(false);
}


```

#### 加密解密


let encodedData = window.btoa("Hello, world"); // 编码
let decodedData = window.atob(encodedData); // 解码

params.passwd = btoa($('input[name="passwd"]').val(), true);

org.apache.commons;

new String(Base64.decodeBase64(form.get("passwd"))));

加密解密账号

```
<script type="text/javascript">
	$(function() {
		$("#btn").click(function() {
			var username = encode64($("#username").val());  //对数据加密
			var password = encode64($("#password").val());
			$("#username").val(username);
			$("#password").val(password);
			document.fm.submit();  //fm为form表单name
		})
	})
	
	// base64加密开始
	var keyStr = "ABCDEFGHIJKLMNOP" + "QRSTUVWXYZabcdef" + "ghijklmnopqrstuv"
			+ "wxyz0123456789+/" + "=";
	
	function encode64(input) {
 
		var output = "";
		var chr1, chr2, chr3 = "";
		var enc1, enc2, enc3, enc4 = "";
		var i = 0;
		do {
			chr1 = input.charCodeAt(i++);
			chr2 = input.charCodeAt(i++);
			chr3 = input.charCodeAt(i++);
			enc1 = chr1 >> 2;
			enc2 = ((chr1 & 3) << 4) | (chr2 >> 4);
			enc3 = ((chr2 & 15) << 2) | (chr3 >> 6);
			enc4 = chr3 & 63;
			if (isNaN(chr2)) {
				enc3 = enc4 = 64;
			} else if (isNaN(chr3)) {
				enc4 = 64;
			}
			output = output + keyStr.charAt(enc1) + keyStr.charAt(enc2)
					+ keyStr.charAt(enc3) + keyStr.charAt(enc4);
			chr1 = chr2 = chr3 = "";
			enc1 = enc2 = enc3 = enc4 = "";
		} while (i < input.length);
 
		return output;
	}
	// base64加密结束
</script>
```



```
private static char[] base64EncodeChars = new char[] { 'A', 'B', 'C', 'D',
		'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q',
		'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', 'a', 'b', 'c', 'd',
		'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q',
		'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', '0', '1', '2', '3',
		'4', '5', '6', '7', '8', '9', '+', '/', };
 
private static byte[] base64DecodeChars = new byte[] { -1, -1, -1, -1, -1,
		-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
		-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
		-1, -1, -1, -1, 62, -1, -1, -1, 63, 52, 53, 54, 55, 56, 57, 58, 59,
		60, 61, -1, -1, -1, -1, -1, -1, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
		10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, -1,
		-1, -1, -1, -1, -1, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37,
		38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, -1, -1, -1,
		-1, -1 };
 
/**
 * 解密
 * @param str
 * @return
 */
public static byte[] decode(String str) {
	byte[] data = str.getBytes();
	int len = data.length;
	ByteArrayOutputStream buf = new ByteArrayOutputStream(len);
	int i = 0;
	int b1, b2, b3, b4;
 
	while (i < len) {
		do {
			b1 = base64DecodeChars[data[i++]];
		} while (i < len && b1 == -1);
		if (b1 == -1) {
			break;
		}
 
		do {
			b2 = base64DecodeChars[data[i++]];
		} while (i < len && b2 == -1);
		if (b2 == -1) {
			break;
		}
		buf.write((int) ((b1 << 2) | ((b2 & 0x30) >>> 4)));
 
		do {
			b3 = data[i++];
			if (b3 == 61) {
				return buf.toByteArray();
			}
			b3 = base64DecodeChars[b3];
		} while (i < len && b3 == -1);
		if (b3 == -1) {
			break;
		}
		buf.write((int) (((b2 & 0x0f) << 4) | ((b3 & 0x3c) >>> 2)));
 
		do {
			b4 = data[i++];
			if (b4 == 61) {
				return buf.toByteArray();
			}
			b4 = base64DecodeChars[b4];
		} while (i < len && b4 == -1);
		if (b4 == -1) {
			break;
		}
		buf.write((int) (((b3 & 0x03) << 6) | b4));
	}
	return buf.toByteArray();
}
```