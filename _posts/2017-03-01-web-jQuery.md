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