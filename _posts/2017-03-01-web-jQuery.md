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
