---
layout: post
title: java中如何遍历集合
date: 2017-02-10 19:33:43
categories: java
tags: javase
author: MarsHu
---

* content
{:toc}

# 遍历java集合 #
java中的集合有List、set、Map集合，分别介绍三种集合的遍历方式。有Person类如下：

    public class Person {
		private String name;
		private int age;

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





## 1.遍历List集合（不加泛型），主要代码如下： ##

> #### 使用传统for循环遍历集合 ####

    public static void main(String[] args) {
		List list = new ArrayList();
		Person person = new Person();
		person.setName("张三");
		person.setAge(25);
		list.add(1);
		list.add("123");
		list.add(person);
		for(int i=0; i<list.size(); i++) {
			Object obj = list.get(i);
			System.out.println(obj.toString());
		}
	}

> #### 使用Lambda表达式遍历集合 ####

    public static void main(String[] args) {
		List list = new ArrayList();
		Person person = new Person();
		person.setName("张三");
		person.setAge(25);
		list.add(1);
		list.add("123");
		list.add(person);
		list.forEach(obj -> System.out.println(obj));
	}

## 2.遍历List集合（有泛型），主要有3种方法，代码如下： ##

> #### 使用传统for循环遍历集合 ####

    public static void main(String[] args) {
		List<String> list = new ArrayList<String>();
		list.add("张三");
		list.add("李四");
		list.add("王五");
		for(int i=0; i<list.size(); i++) {
			String str = list.get(i);
			System.out.println(str);
		}
	}

> #### 使用增强for循环遍历集合 ####

    public static void main(String[] args) {
		List<String> list = new ArrayList<String>();
		list.add("张三");
		list.add("李四");
		list.add("王五");
		for(String str : list) {
			System.out.println(str);
		}
	}

> #### 使用Lambda表达式遍历集合 ####

    public static void main(String[] args) {
		List<String> list = new ArrayList<String>();
		list.add("张三");
		list.add("李四");
		list.add("王五");
		list.forEach(str -> System.out.println(str));
		/*单个内容输出 */
		list.forEach(str -> {
			if ("张三".equals(str)) {
				System.out.println(str);
			}
		});
	}

## 3.遍历Set集合（不加泛型），代码如下 ##

> #### 使用Iterator接口遍历Set集合 ####

    public static void main(String[] args) {
		Set set =new HashSet();
		Person person = new Person();
		person.setName("张三");
		person.setAge(25);
		set.add(1);
		set.add("123");
		set.add(person);
		Iterator it = set.iterator();
		while(it.hasNext()) {
			Object obj = it.next();
			System.out.println(obj.toString());
		}
	}

> #### 使用Lambda表达式遍历Set集合 ####

    public static void main(String[] args) {
		Set set =new HashSet();
		Person person = new Person();
		person.setName("张三");
		person.setAge(25);
		set.add(1);
		set.add("123");
		set.add(person);
		set.forEach(obj -> System.out.println(obj));
	}

## 4.遍历Set集合（有泛型），代码如下 ##

> #### 使用Iterator接口遍历Set集合 ####
    
    public static void main(String[] args) {
		Set<String> set =new HashSet<String>();
		set.add("张三");
		set.add("李四");
		set.add("王五");
		Iterator<String> it = set.iterator();
		while(it.hasNext()) {
			String str = it.next();
			System.out.println(str);
		}
	}

> #### 使用Lambda表达式遍历Set集合 ####

    public static void main(String[] args) {
		Set<String> set =new HashSet<String>();
		set.add("张三");
		set.add("李四");
		set.add("王五");
		set.forEach(str -> System.out.println(str));
	}

## 5.遍历Map集合（不加泛型），主要代码如下： ##

> #### 遍历key的方式遍历Map集合 ####

    public static void main(String[] args) {
		Map map = new HashMap();
		Person person = new Person();
		person.setName("张三");
		person.setAge(25);
		map.put(1, 1);
		map.put("123", "123");
		map.put(person, person);
		Set keySet = map.keySet();
		for (Iterator i = keySet.iterator(); i.hasNext();) {
			Object key = i.next();
			Object value = map.get(key);
			System.out.println(key + ":" + value);
		}
	}

> #### 遍历Entry的方式遍历Map集合 ####

    public static void main(String[] args) {
		Map map = new HashMap();
		Person person = new Person();
		person.setName("张三");
		person.setAge(25);
		map.put(1, 1);
		map.put("123", "123");
		map.put(person, person);
		Set<Entry> entries=map.entrySet();
		for (Entry e : entries) {
		    Object key = e.getKey();
		    Object value = e.getValue();
		    System.out.println(key+":"+value);
		 }
	}

> #### 使用Lambda表达式遍历Map集合 ####

    public static void main(String[] args) {
		Map map = new HashMap();
		Person person = new Person();
		person.setName("张三");
		person.setAge(25);
		map.put(1, 1);
		map.put("123", "123");
		map.put(person, person);
		map.forEach((key, value) -> System.out.println(key + ":" + value));
	}

## 6.遍历Map集合（有泛型），主要代码如下： ##

> #### 遍历key的方式遍历Map集合 ####

    public static void main(String[] args) {
		Map<String, String> map = new HashMap<String, String>();
		map.put("张三", "张三");
		map.put("李四", "李四");
		map.put("王五", "王五");
		Set<String> keySet = map.keySet();
		for (Iterator<String> i = keySet.iterator(); i.hasNext();) {
			String key = i.next();
			String value = map.get(key);
			System.out.println(key + ":" + value);
		}
	}

> #### 遍历Entry的方式遍历Map集合 ####

    public static void main(String[] args) {
		Map<String, String> map = new HashMap<String, String>();
		map.put("张三", "张三");
		map.put("李四", "李四");
		map.put("王五", "王五");
		Set<Entry<String, String>> entries = map.entrySet();
		for (Entry<String, String> e : entries) {
			String key = e.getKey();
			String value = e.getValue();
			System.out.println(key + ":" + value);
		}
	}

> #### 使用Lambda表达式遍历Map集合 ####

    public static void main(String[] args) {
		Map<String, String> map = new HashMap<String, String>();
		map.put("张三", "张三");
		map.put("李四", "李四");
		map.put("王五", "王五");
		map.forEach((key, value) -> System.out.println(key + ":" + value));
	}