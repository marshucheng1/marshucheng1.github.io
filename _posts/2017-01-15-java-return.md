---
layout: post
title: return、break、continue的区别
date: 2017-01-15 14:36:15
categories: java
tags: javabasic
author: MarsHu
---

* content
{:toc}

# Java中关键字continue、break和return的区别 #

> continue：跳出本次循环继续下一次循环。
> 
> break：   跳出循环体，继续执行循环外的函数体。
> 
> return：  跳出整个函数体，函数体后面的部分不再执行。





请看如下代码:
```
public static void main(String[] args) {
        int j=3;
        for(int i=0;i<5;i++){
            if(i==j){
                代码部分;
            }
            System.out.println("i="+i);
        }
        System.out.println("循环结束...");
 }
```
1. 代码部分为continue时，打印结果为：
```
i=0
i=1
i=2
i=4
循环结束...
```
2. 代码部分为break时，打印结果为：
```
i=0
i=1
i=2
循环结束...
```
3. 代码部分为return时，打印结果为：
```
i=0
i=1
i=2
```