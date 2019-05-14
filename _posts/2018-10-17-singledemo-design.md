---
layout: post
title: java设计模式-单例设计模式
date: 2018-10-17 23:00:00
categories: java
tags: design
author: MarsHu
---

* content
{:toc}

### 单例设计模式 ###
解决问题：可以保证一个类在内存中的对象唯一性。

应用场景：
两个应用程序，`A`和`B`，都会用到同一个配置信息`Configue`。假设`Configue`对象中有2个属性，`username`，`password`。

当我们在`A`程序中使用`Configue`时，并对`Configue`的`username`和`password`属性做出了修改。
此时我们在`B`程序中使用`Configue`时，我们需要反应出`A`程序中修改后的结果，而不是初始结果。

解决这种方案时，我们就要使用到单例模式了。





### 解决思路 ###
1、不允许其他程序用`new`创建该类对象

2、在该类中创建一个本类实例

3、对外提供一个方法，让其他程序可以获取第2步中创建的对象

### 实现步骤 ###
1、私有化该类的构造函数

2、通过`new`在本类中创建一个本类对象

3、定义一个`public`方法，将创建的对象返回

### 单例代码 ###

	public class SingleDemo {
		/** 创建一个本类对象 */
		SingleDemo singleDemo = new SingleDemo();
		/** 私有化构造函数 */
		private SingleDemo() {
		}
		/** 公共的获取对象方法 */
		public SingleDemo getInstance() {
			return singleDemo;
		}
	}

### 使用上面的单例 ###

	public class SingleTest {
		public static void main(String[] args) {
			new SingleDemo().getInstance();
		}
	}

当我们想使用创建的单例时，发现了一些问题。我们无法通过上述方式获取了。这个时候我们需要修改单例代码。

	public class SingleDemo {
		/** 创建一个本类对象 */
		static SingleDemo singleDemo = new SingleDemo();
		/** 私有化构造函数 */
		private SingleDemo() {
		}
		/** 公共的获取对象方法 */
		public static SingleDemo getInstance() {
			return singleDemo;
		}
	}

此时我们同时调整使用单例的代码

	public class SingleTest {
		public static void main(String[] args) {
			SingleDemo singleDemo = SingleDemo.getInstance();
		}
	}

当大家在写完这段代码之后，又会发现一个新的问题，那就是既然`SingleDemo`是`static`成员变量，我同样可以直接通过`.singleDemo`的方式获取，如下：

	public class SingleTest {
		public static void main(String[] args) {
			//SingleDemo singleDemo = SingleDemo.getInstance();
			SingleDemo singleDemo = SingleDemo.singleDemo;
		}
	}

其实这种方式就和我们所用的成员属性加上`get`、`set`方法原理一样。通过方法获取对象，更加可控。

为了避免直接访问属性获取对象，可以对单例进行如下修改：

	public class SingleDemo {
		/** 创建一个本类对象 */
		private static SingleDemo singleDemo = new SingleDemo();
		/** 私有化构造函数 */
		private SingleDemo() {
		}
		/** 公共的获取对象方法 */
		public static SingleDemo getInstance() {
			return singleDemo;
		}
	}

加上`private`修饰后，就无法再通过属性直接访问了。

此时我们再次修改测试代码，来测试获取的对象是否唯一。

	public class SingleTest {
		public static void main(String[] args) {
			//SingleDemo singleDemo = SingleDemo.getInstance();
			//SingleDemo singleDemo = SingleDemo.singleDemo;
			SingleDemo s1 = SingleDemo.getInstance();
			SingleDemo s2 = SingleDemo.getInstance();
			//打印结果是true
			System.out.println(s1 == s2);
		}
	}


### 简单案例 ###
上面的例子只是简单的单例实现过程，并没有什么实际使用意义。简单写一个`demo`演示。

单例`demo`

	public class SingleFoo {
		private int num;
		
		private static SingleFoo singleFoo = new SingleFoo();
		
		private SingleFoo() {
		}
		
		public static SingleFoo getInstance() {
			return singleFoo;
		}
	
		public int getNum() {
			return num;
		}
	
		public void setNum(int num) {
			this.num = num;
		}
		
	}

测试`demo`

	public class SingleTest {
		public static void main(String[] args) {
			SingleFoo s1 = SingleFoo.getInstance();
			SingleFoo s2 = SingleFoo.getInstance();
			s1.setNum(10);
			s2.setNum(20);
			/** 结果是20 */
			System.out.println(s1.getNum());
			/** 结果是20 */
			System.out.println(s2.getNum());
		}
	}


### 饿汉式和懒汉式 ###
根据单例对象获取时机的不同，可以将单例模式分为饿汉式和懒汉式。多线程下没有隐患

饿汉式：类一加载，对象就已经存在于内存[开发中常用]

	public class SingleDemo {
		private static SingleDemo s = new SingleDemo();
		private SingleDemo() {
		}
		public static SingleDemo getInstance() {
			return s;
		}
	}

懒汉式：类加载进来，没有对象。只有调用getInstance()方法，才会创建对象。[面试常用]。多线程下有安全隐患。

	public class SingleDemo {
		private static SingleDemo s = null;
		private SingleDemo() {
		}
		public static SingleDemo getInstance() {
			if (s == null) {
				s = new SingleDemo();
			}
			return s;
		}
	}

### 多线程下的单例 ###
多线程下，懒汉式会有安全隐患，所以需要改造一下。

第一种：`synchronized`关键字修饰`getIntance`方法

	public class SingleDemo {
		private static SingleDemo s = null;
		private SingleDemo() {
		}
		public static synchronized SingleDemo getInstance() {
			if (s == null) {
				s = new SingleDemo();
			}
			return s;
		}
	}

这种方式，在实际使用中，效率并不高，因为多线程下，需要等待访问。

第二种：`synchronized`关键字修饰`getIntance`方法内的代码块。

	public class SingleDemo {
		private static SingleDemo s = null;
		private SingleDemo() {
		}
		public static SingleDemo getInstance() {
			synchronized(SingleDemo.class) {
				if (s == null) {
					s = new SingleDemo();
				}
			}
			return s;
		}
	}

这种方式，成功避免了多线程下等待访问`getInstance`方法，但是同样会等待访问`if`代码块

第三种：将`if`判断提前，这样就会避免多次停留在等待访问。

	public class SingleDemo {
		private static SingleDemo s = null;
		private SingleDemo() {
		}
		public static SingleDemo getInstance() {
			if (s == null) {
				synchronized(SingleDemo.class) {
					if (s == null) {
						s = new SingleDemo();
					}
				}
			}
			return s;
		}
	}

