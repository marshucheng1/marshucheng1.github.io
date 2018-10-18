---
layout: post
title: BigDecimal不丢失精度
date: 2017-01-02 08:33:26
categories: java
tags: javabasic
author: MarsHu
---

* content
{:toc}

# BigDecimal——不丢失精度之加减乘除 #

----------

1. Java在直接进行浮点型运算时，比较容易出现精度丢失的问题，导致意料之外的运算结果；可以使用BigDecimal则提供的方法避免精度丢失。
2. 产生精度丢失的原因是二进制不能精确的表示1/10，如同十进制不能精确的表示1/3一样。
3. 直接进行浮点运算，代码如下
```
public class Demo {
	public static void main(String[] args) {
		test();
	}
	public static void test() {
		System.out.println("加法结果："+(1.1+2.2));//3.3
		System.out.println("减法结果："+(2.2-10.1));//-7.9
		System.out.println("乘法结果："+(1.1*2.2));//2.42
		System.out.println("除法结果："+(4.4/10));//0.44
}
```




4. 使用BigDecimal进行浮点型运算
```
public class BigDecimalDemo{
	public static void main(String[] args) {
		test();
	}
	public static void test() {
		//构造函数的参数是String类型
		//加法
		BigDecimal bigDecimal1 = new BigDecimal(String.valueOf(1.1));
		BigDecimal bigDecimal2 = new BigDecimal(String.valueOf(2.2));
		String str1 = bigDecimal1.add(bigDecimal2).toString();
		//减法
		bigDecimal1 = new BigDecimal(String.valueOf(2.2));
		bigDecimal2 = new BigDecimal(String.valueOf(10.1));
		String str2 = bigDecimal1.subtract(bigDecimal2).toString();
		//乘法
		bigDecimal1 = new BigDecimal(String.valueOf(1.1));
		bigDecimal2 = new BigDecimal(String.valueOf(2.2));
		String str3 = bigDecimal1.multiply(bigDecimal2).toString();
		//除法
		bigDecimal1 = new BigDecimal(String.valueOf(4.4));
		bigDecimal2 = new BigDecimal(String.valueOf(10));
		String str4 = bigDecimal1.divide(bigDecimal2).toString();
		//结果
		System.out.println("加法结果："+Double.valueOf(str1));
		System.out.println("减法结果："+Double.valueOf(str2));
		System.out.println("乘法结果："+Double.valueOf(str3));
		System.out.println("除法结果："+Double.valueOf(str4));
	}
}
```

----------
运行的结果如下：
```
加法结果：3.3
减法结果：-7.9
乘法结果：2.42
除法结果：0.44
```
