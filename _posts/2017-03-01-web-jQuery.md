---
layout: post
title: 前端相关知识-js、jQuery、css
date: 2017-03-01 13:31:22
categories: WEB
tags: jQuery
author: MarsHu
---

* content
{:toc}

### 页面加载完成后执行 ###
	第一种：
    $(document).ready(function () {
		......
    }

	第二种：
    $(function () {
		......
	});

### 检查内容是否超出设定宽度 ###

	this指代当前需要判断的对象
	this.offsetWidth < this.scrollWidth





### 获取图片的src属性值 ###

	$("#imgId")[0].src; 

### 修改图片的src属性值 ###

	$("#imgId").attr('src', path); 

### css内容超出宽度显示... ###

    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;

### checkbox设置不能勾选 ###

	disabled="disabled"

### 元素定位后垂直居中 ###
假定该元素上一个元素的高度是`460px`，这里`margin-left`的值整合是宽度的一半。顶部偏移：是`460/2-110/2=175`。

	position: absolute;
	width: 102px;
	height: 110px;
	left: 50%;
	margin-left: -51px;
	top: 175px;

### javaScript原生ajax请求-页面加载完执行 ###

	function myfun() {
        if (window.XMLHttpRequest) {
            var oAjax = new XMLHttpRequest();
        } else {
            var oAjax = new ActiveXObject("Microsoft.XMLHTTP");
        }
        oAjax.open("GET", "xxx请求地址?t='+new Date().getTime()", true);
        oAjax.send();
        oAjax.onreadystatechange = function () {
            if (oAjax.readyState == 4) {
                if (oAjax.status == 200) {
					console.log("success");
                    console.log(oAjax.responseText);
                } else {
                    console.log("error");
                }
            }
        };
    }
    window.onload = myfun;

### javaScript获取当前地址栏url ###

	var href = window.location.href;

### javaScript刷新页面 ###
	
	window.location.reload()

### 以下载的形式打开txt类型文件 ###
txt类型文件，在浏览器中默认是直接打开的，通过设置可以实现直接下载的形式。

原文地址：`https://www.jianshu.com/p/d3378320289c`。具体代码如下，省略部分代码

	......
	success: function (data) {
		console.log(data);
		var url = config.baseDataUrl + "/" + downname;
		//window.location.href = url;
		const fileName = url.split('/').pop().split('?')[0];
		var a = document.createElement('a');
		a.download = fileName;
		a.href = url;
		document.body.appendChild(a);
		a.click();
		document.body.removeChild(a);
		window.URL.revokeObjectURL(url);
	}
	......

### 禁止textarea拖动 ###
textarea默认情况可以拖拉改变大小，会影响显示效果，追加`resize: none`样式，可以禁止拖拉。代码如下

	<textarea style="resize: none" rows="10"></textarea>

### 微信公众号页面自动关闭 ###
实际开发过程中，在获取openid或有其他业务时，需要实现页面自动关闭。代码如下：

	<!DOCTYPE html>
	<html>
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title>one</title>
	</head>
	<body>
	
	</body>
	<script>
		setTimeout(function() {
			//这个可以关闭安卓系统的手机
			document.addEventListener('WeixinJSBridgeReady', function() {
				WeixinJSBridge.call('closeWindow');
			}, false);
			//这个可以关闭ios系统的手机
			WeixinJSBridge.call('closeWindow');
		}, 1000)
	</script>

### JS实现给数组对象排序 ###
假设有一个对象数组，我们想要根据某个对象属性对数组进行排序。而传递给数组`sort()`方法的比较函数要接收2个参数，即要比较的值。

可是，我们需要一种方式指明按照哪个属性来排序。要解决这个问题，可以定义一个函数，它接收一个属性名，然后根据这个属性名来创建一个比较函数。

	function createComprisonFunction(propertyName){
	    return function(object1,object2){
	        var value1 = object1[propertyName];
	        var value2 = object2[propertyName];
	        if(value1 < value2){
	            return -1;
	        }else if(value1 > value2){
	            return 1;
	        }else{
	            return 0;
	        }
	    }
	}
	//上面的函数可以像在下面的列子中这样使用
	var data = [{name:"Zachary",age:28},{name:"Nicholas",age:29}];
	data.sort(createComprisonFunction("name"));
	console.log(data[0].name); //Nicholas
	data.sort(createComprisonFunction("age"));
	console.log(data[0].name); //Zachary