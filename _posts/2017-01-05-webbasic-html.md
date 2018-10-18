---
layout: post
title: html标签中的lang属性
date: 2017-01-05 13:31:27
categories: webbasic
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
`<html lang="en">`
```
这里的lang="en"可以删除，如果不删除，用谷歌之类的浏览器打开，它会认为是英文的，会提示翻译。




4. 常见语言类型。
```
`<html lang='en'></html>`#英文
`<html lang='zh'></html>`#中文
`<html lang='ja'></html>`#日文
`<html lang='en-us'></html>`#美式英文
```
5. lang属性中的语言代码不区分大小写。
```
`<html lang='en-us'></html>`#英文
`<html lang='en-US'></html>`#英文
```
上面的两行代码一样的效果。
6. lang属性也可以加到普通标签上，如
```
`<div lang='en'>this is English .</div>`
```