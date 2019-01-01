---
layout: post
title: 浅谈JAVA反射机制
date: 2017-02-12 17:33:43
categories: java
tags: javase
author: MarsHu
---

* content
{:toc}

# 什么是JAVA反射机制 #
反射是JAVA的API。它允许程序在运行过程中取得任何一个已知名称的类的内部信息，包括其中的构造方法、声明的字段和定义的方法等。

（这个定义非常有趣，我们可以想到我们所学的框架、导入的jar包都运用到了反射）。

利用反射API我们可以实现动态执行：

- 动态加载类，获取类信息

- 动态创建类的对象

- 动态访问类的属性

- 动态调用类的方法





# JAVA的动态、静态怎么理解 #
我们在学习面向对象时，经常会说到编译时类型和运行时类型两个名词。例如代码：`Person p = new Student();`。

这行代码的p变量，在编译时的类型为Person，运行时的类型为Student。关于这个大家应该都不会陌生。

我们喜欢称之为多态。现在我们再来说说动态执行和静态执行。

> 动态执行：只在JVM运行期间才确定的执行次序。
	
	Class cls = Class.forName(类名);

这行代码我们大家都不陌生，在学习jdbc时，经常用到。事先并不知道一定要使用哪种数据库的驱动。

运行时，动态输入类名，可以是任何类名，然后获取该类的相关信息。

这行代码叫做动态加载类，后面会介绍。这种JVM运行，提供输入，才确定执行的过程，叫做动态执行。

> 静态执行：编译后就确定程序的执行次序，JVM运行期按照既定的次序执行。
	
	Foo foo = new Foo();
	foo.hello();

这两行代码就是典型的静态执行。



# 反射的主要功能 #
## 动态加载类 ##

	Class cls = Class.forName(类名);

这行代码的作用是动态加载类到内存方法区。


> 在Java中，每个class都有一个相应的Class对象。也就是说，当我们编写一个类，
> 
> 编译完成后，在生成的.class文件中，就会产生一个Class对象，用于表示这个类的类型信息。
> 
> 关于方法区这个概念，1.6，1.7可以将方法区看成永久代，1.8之后，永久代取消，
> 
> 替代的是元数据区，也渐渐没有方法区的提法了，当然我们仍然可以像1.6,1.7那样
> 
> 将方法区看成元数据区。鉴于中国大环境，1.6，1.7仍然是主流版本。

这段代码我们可以得到以下几个信息：

- 类名是动态输入的，可以是任何类名。输入时请加上包名。例：`包名.类名`的形式

- cls引用指向的对象，可以访问方法区中对应的类的信息

- 如果输入了错误的类名，或者是不存在的类名，将会出现一个最常见的异常`ClassNotFoundException`

假定动态输入的类为`demo包`下的`Person类`。

（当然你也可以用java提供的类，例如：`java.util.Date`,`java.util.ArrayList`等,后面的调用有参数构造函数，请自行修改）

完整代码如下：

	package demo;

	public class Person {
		private String name;
		private int age;
	
		public Person() {
		}

		public Person(String name, int age) {
			this.name = name;
			this.age = age;
		}

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}

		public int getAge() {
			return age;
		}

		public void setAge(int age) {
			this.age = age;
		}

		public String toString() {
			return "Person [name=" + name + ", age=" + age + "]";
		}
	}


完整代码

	public class ReflectDemo {
		public static void main(String[] args) throws Exception {
			Scanner sc = new Scanner(System.in);
			System.out.println("请输入类名:");
			String className = sc.nextLine();
			Class cls = Class.forName(className);
			System.out.println(cls);
		}
	}

## 动态创建对象 ##
Class提供了动态创建对象的方法：`Object newInstance()`

使用该方法需要注意以下几点：

- 该方法调用类的无参构造函数创建对象，如果没有无参构造函数，将抛出没有方法的异常`NoSuchMethodException`

- 因为返回值需要指代任意对象，所以类型为Object

- 如果需要调用有参数构造函数，可以利用Constructor类的方法实现。但是这种方式不灵活。

完整代码：

	public class ReflectDemo {
		public static void main(String[] args) throws Exception {
			Scanner sc = new Scanner(System.in);
			System.out.println("请输入类名:");
			String className = sc.nextLine();
			Class cls = Class.forName(className);
			System.out.println(cls);
			/*调用无参构造函数创建对象*/
			Object obj = cls.newInstance();
			System.out.println(obj);
			/*调用有参数构造函数创建对象*/
			/*创建Constructor对象*/
			Constructor con = cls.getConstructor(String.class, int.class);
			/*通过带参构造函数创建对象*/
			Object obj2 = con.newInstance("张三",25);
			System.out.println(obj2);
		}
	}

## 动态调用方法 ##
假定动态输入的类为`demo包`下的`Demo类`,完整代码如下：

	package demo;

	public class Demo {
	
		public String sayHi(String name) {
			System.out.println("hello"+name);
			return "hello" + name;
		}
	
		public void testGetName() {
			System.out.println("getname");
		}
	
		public void testGetAge() {
			System.out.println("getage");
		}
	
		public void testGetEmail() {
			System.out.println("getEmail");
		}
	
		public int printInfo(int num) {
			System.out.println("DemoInfo"+num);
			return num;
		}
	}

> #### 现在我们有个需求，我们想要执行全部以test为开头的方法 ####

想要实现这个需求，如果我们不利用反射机制相关的API是没办法做到的。我们可以通过如下具体方式实现这个需求。
### 动态发现方法 ###
我们可以使用Class类提供的`Method[] getDeclaredMethods()`方法获取所有的方法信息。
 
- `Method`代表方法信息，可以使用Method类提供的API获取方法详细信息，如：方法名、返回值类型、参数列表等。
- `Method[]`返回数组代表当前类中全部方法，每一个元素对应一个方法。

完整代码如下：

	package demo;

	import java.lang.reflect.Method;
	import java.util.Scanner;

	public class ReflectDemo {
		public static void main(String[] args) throws Exception {
			Scanner sc = new Scanner(System.in);
			System.out.println("请输入类名:");
			String className = sc.nextLine();
			Class cls = Class.forName(className);
			System.out.println(cls);
			Object obj = cls.newInstance();
			System.out.println(obj);
			/*获取类的全部方法信息*/
			Method[] methods = cls.getDeclaredMethods();
			/*输出每一个方法的具体信息*/
			for (Method method : methods) {
				System.out.println(method);
				/*获取方法名*/
				System.out.println(method.getName());
				/*获取方法返回值*/
				System.out.println(method.getReturnType());
			}
		}
	}

### 动态执行方法 ###
Method类提供的`Object invoke(Object obj, Object... args)`方法，可以帮助我们动态执行方法。

 - obj代表一个对象，该对象上一定包含当前方法，否则调用会出现异常。
 - args代表调用方法时，传递的实际参数，如果没有参数可以不用或者传递null。如果传入参数，一定
 
	要注意参数的个数和类型需要匹配，否则出现异常。
 - 因为可以是任意类型的返回值，所以用Object，调用没有返回值方法时，返回值为null。

执行以test为开头的方法：

	package demo;

	import java.lang.reflect.Method;
	import java.util.Scanner;

	public class ReflectDemo {
		public static void main(String[] args) throws Exception {
			Scanner sc = new Scanner(System.in);
			System.out.println("请输入类名:");
			String className = sc.nextLine();
			Class cls = Class.forName(className);
			System.out.println(cls);
			Object obj = cls.newInstance();
			System.out.println(obj);
			/* 获取类的全部方法信息 */
			Method[] methods = cls.getDeclaredMethods();
			/* 输出每一个方法的具体信息 */
			for (Method method : methods) {
				/* 运行以test为开头的方法 */
				if (method.getName().startsWith("test")) {
					method.invoke(obj);
				}
			}
		}
	}

上面的代码只能执行没有参数的方法，如果方法有参数时，必须明确定义参数：

	package demo;

	import java.lang.reflect.Method;
	import java.util.Scanner;

	public class ReflectDemo {
		public static void main(String[] args) throws Exception {
			Scanner sc = new Scanner(System.in);
			System.out.println("请输入类名:");
			String className = sc.nextLine();
			Class cls = Class.forName(className);
			System.out.println(cls);
			Object obj = cls.newInstance();
			System.out.println(obj);
			/* 获取类的全部方法信息 */
			Method[] methods = cls.getDeclaredMethods();
			/* 输出每一个方法的具体信息 */
			for (Method method : methods) {
				/*指定运行sayHi方法*/
				if (method.getName().startsWith("say")) {
					method.invoke(obj, "张三");
				}
				/*指定运行printInfo方法*/
				if (method.getName().startsWith("print")) {
					method.invoke(obj, 35);
				}
			}
		}
	}

最后，我们一起来看看一张反射的内存图解（水平有限，有错之处，多多包涵）。
![reflect.png](http://marshucheng1.github.io/assets/reflect.png)