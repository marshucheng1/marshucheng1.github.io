---
layout: post
title: html标签中的lang属性
date: 2017-01-05 13:31:27
categories: WEB
tags: html
author: MarsHu
---

* content
{:toc}

# lang属性解释 #
----------
1. lang 属性规定元素内容的语言。
2. 在 HTML5 中， lang 属性可用于任何的 HTML 元素 (它会验证任何HTML元素。但不一定是有用)。
3. 声明当前页面的语言类型
```
	<html lang="en">
```
这里的`lang="en"`可以删除，如果不删除，用谷歌之类的浏览器打开，它会认为是英文的，会提示翻译。




4. 常见语言类型。
```
	<html lang='en'></html>#英文
	<html lang='zh'></html>#中文
	<html lang='ja'></html>#日文
	<html lang='en-us'></html>#美式英文
```
5. lang属性中的语言代码不区分大小写。
```
	<html lang='en-us'></html>#英文
	<html lang='en-US'></html>#英文
```
上面的两行代码一样的效果。
6. lang属性也可以加到普通标签上，如
	```
	<div lang='en'>this is English .</div>
	```
表示该div内部内容为英文

7.font综合属性的设置顺序

	font:font-style font-variant font-weight font-size/line-height font-family

	font-style:字体风格（倾斜）
	font-variant：字母大小写
	font-weight：加粗
	font-size：字体大小
	font-family：何种字体

	1.只有同时设置font-size和font-family，属性才会起作用
	2.值之间空格隔开
	3.前面3个属性可以任意顺序，后面2个必须固定

8.background综合属性设置顺序

	background:background-color background-image background-repeat background-attachment background-position 

	background-color：背景颜色
	background-image：背景图片
	background-repeat：是否重复
	background-attachment：是否随滚轮滚动
	background-postion：图片位置

	1.各值之间用空格分割，不分先后顺序
	2.背景图片会遮盖背景色

9.list-style综合属性设置顺序

	list-style:list-style-type list-style-position list-style-image

	1.值之间用空格分隔
	2.顺序可以不固定
	3.list-style-image会覆盖list-style-type的设置

10.内联元素之间的缝隙

	<span>内联元素1111</span>
	<span>内联元素2222</span>
	<span>内联元素3333</span>

这是因为元素之间是有换行的，想要清除之间的缝隙，可以选择写在一行。

	<span>内联元素1111</span><span>内联元素2222</span><span>内联元素3333</span>

还可以使用1个div将内联元素包括。

	<div>
		<span>内联元素1111</span>
		<span>内联元素2222</span>
		<span>内联元素3333</span>
	</div>

	通过设置div的字体大小为0，然后再设置span的字体大小为16px

	<style>
		div{font-size:0px};span{font-size:16px}
	</style>

11.清除浮动的方法

a.在浮动元素后使用一个空元素
	
	<style>
		.clear{clear:both}
	</style>
	<div class="clear"></div>

b.给浮动元素的容器添加 overflow:hidden

	兼容低版本IE
	zoom:1;/*触发haslayout兼容IE 6、7*/

c.使用CSS3的:after伪元素

给浮动元素的容器添加一个类clearfix

	.clearfix:after{
		content:".";
		display:block;
		height:0;
		visibility:hidden;
		clear:both;
	}
	
	兼容低版本IE
	.clearfix{zoom:1;} /*触发haslayout兼容IE 6、7*/

d.父级元素定义height。只适合高度固定的布局。

e.父级元素也一起浮动。不推荐，会产生新的浮动问题。