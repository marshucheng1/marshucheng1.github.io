---
layout: post
title: 使用Spring MVC开发RESTful API（二）
date: 2020-01-02 11:31:22
categories: JavaWeb
tags: springboot
author: MarsHu
---

* content
{:toc}

### RESTFUL API介绍 ###
第一印象：

	传统请求方式：
	查询	/user/query?name=tome		GET
	详情	/user/getInfo?id=1		GET
	创建	/user/create?name=tom		POST
	修改	/user/update?id=1&name=jerry	POST
	删除	/user/delete?id=1		GET
	RESTful API请求方式：
	查询	/user?name=tome			GET
	详情	/user/1				GET
	创建	/user				POST
	修改	/user/1				PUT
	删除	/user/1				DELETE

RESTFUL API说明：
1.用URL描述资源。

2.使用HTTP方法描述行为。使用HTTP状态码来表示不同结果。

3.使用json交互数据。

4.RESTFUL只是一种风格，并不是一种强制的标准。






### 一、使用springboot测试框架，编写测试用例--查询请求 ###
引入`spring-boot-starter-test`。修改`zhqx-security-demo`模块的pom文件

	<project xmlns="http://maven.apache.org/POM/4.0.0"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<artifactId>zhqx-security-demo</artifactId>
		<parent>
			<groupId>com.zhqx.security</groupId>
			<artifactId>zhqx-security</artifactId>
			<version>1.0.0-SNAPSHOT</version>
			<relativePath>../zhqx-security</relativePath>
		</parent>
	
		<dependencies>
			<dependency>
				<groupId>com.zhqx.security</groupId>
				<artifactId>zhqx-security-browser</artifactId>
				<version>${zhqx.security.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-test</artifactId>
			</dependency>
		</dependencies>
	</project>

创建测试用例：

伪造WEB-MVC的运行环境，方便测试。需要引入`WebApplicationContext`以及`MockMvc`。

说明：

`WebApplicationContext`,是继承于`ApplicationContext`的一个接口，扩展了`ApplicationContext`，是专门为Web应用准备的，它允许从相对于Web根目录的路径中装载配置文件完成初始化。

`MockMvc`实现了对Http请求的模拟，能够直接使用网络的形式，转换到Controller的调用，这样可以使得测试速度快、不依赖网络环境，而且提供了一套验证的工具，这样可以使得请求的验证统一而且很方便。

测试用例代码如下：

	package com.zhqx.web.controller;
	
	import org.junit.Before;
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.context.SpringBootTest;
	import org.springframework.http.MediaType;
	import org.springframework.test.context.junit4.SpringRunner;
	import org.springframework.test.web.servlet.MockMvc;
	import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
	import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
	import org.springframework.test.web.servlet.setup.MockMvcBuilders;
	import org.springframework.web.context.WebApplicationContext;
	
	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class UserControllerTest {
		
		@Autowired
		private WebApplicationContext wac;
	
		private MockMvc mockMvc;
		
		@Before
		public void setup() {
			mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
		}
		
		@Test
		public void whenQuerySuccess() throws Exception {
			mockMvc.perform(MockMvcRequestBuilders.get("/user")
					.contentType(MediaType.APPLICATION_JSON_UTF8))//请求的数据格式是json
					.andExpect(MockMvcResultMatchers.status().isOk())//期待返回的状态码是200(即正常)
					.andExpect(MockMvcResultMatchers.jsonPath("$.length()").value(3));//期待返回数组的长度是3
		}
	}

因为，在后续的测试中，我们会频繁用到`MockMvcRequestBuilders`和`MockMvcResultMatchers`类中的静态方法，为了避免后面反复声明这2个类。

我们可以将这2个类加入STS的偏好设置中。设置路径为：`Window->Preferences->Java->Editor->Content Assist->Favorites`

进入设置目录后，点击`New Type...`然后选择`Browse...`，依次添加`MockMvcRequestBuilders`和`MockMvcResultMatchers`

修改测试用例代码，如下：

	package com.zhqx.web.controller;
	
	import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
	import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
	import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
	
	import org.junit.Before;
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.context.SpringBootTest;
	import org.springframework.http.MediaType;
	import org.springframework.test.context.junit4.SpringRunner;
	import org.springframework.test.web.servlet.MockMvc;
	import org.springframework.test.web.servlet.setup.MockMvcBuilders;
	import org.springframework.web.context.WebApplicationContext;
	
	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class UserControllerTest {
		
		@Autowired
		private WebApplicationContext wac;
	
		private MockMvc mockMvc;
		
		@Before
		public void setup() {
			mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
		}
		
		@Test
		public void whenQuerySuccess() throws Exception {
			mockMvc.perform(get("/user")
					.contentType(MediaType.APPLICATION_JSON_UTF8))//请求的数据格式是json
					.andExpect(status().isOk())//期待返回的状态码是200(即正常)
					.andExpect(jsonPath("$.length()").value(3));//期待返回集合的长度是3
		}
	}

此时运行测试用例，显示失败，提示出现404错误，因为还没有写UserController。

> **编写UserController，使用RESTful api**

	UserController代码如下：
	
	package com.zhqx.web.controller;
	
	import java.util.List;
	
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;
	import org.springframework.web.bind.annotation.RestController;
	
	import com.zhqx.dto.User;
	
	@RestController
	public class UserController {
		
		@RequestMapping(value = "/user", method = RequestMethod.GET)
		public List<User> query() {
			return null;
		}
	}

User类的代码如下：

	package com.zhqx.dto;
	
	public class User {
		private String username;
		
		private String password;
	
		public String getUsername() {
			return username;
		}
	
		public void setUsername(String username) {
			this.username = username;
		}
	
		public String getPassword() {
			return password;
		}
	
		public void setPassword(String password) {
			this.password = password;
		}
		
	}

此时再次运行测试用例，依然出错，错误信息是说jsonPath没有内容。

调整代码，修改UserController类，修改`query()`方法添加三个User实例。

	@RequestMapping(value = "/user", method = RequestMethod.GET)
	public List<User> query() {
		List<User> users = new ArrayList<User>();
		users.add(new User());
		users.add(new User());
		users.add(new User());
		return users;
	}

此时再次运行测试用例，测试结果正常。

> **添加查询参数以及测试用例如何带参数测试**

使用`@RequestParam`注解添加查询参数。修改`query()`方法:

	@RequestMapping(value = "/user", method = RequestMethod.GET)
	public List<User> query(@RequestParam String username) {
		System.out.println(username);
		List<User> users = new ArrayList<User>();
		users.add(new User());
		users.add(new User());
		users.add(new User());
		return users;
	}

代表查询时需要有username参数，如果没有的话，会报错。此时运行测试用例，会报错400，表示请求格式错误。所以我们需要传入参数。
修改`UserControllerTest`，修改`whenQuerySuccess()`方法：

	@Test
	public void whenQuerySuccess() throws Exception {
		mockMvc.perform(get("/user")
				.param("username", "jojo")
				.contentType(MediaType.APPLICATION_JSON_UTF8))//请求的数据格式是json
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.length()").value(3));//期待返回集合的长度是3
	}

> **@RequestParam注解的属性说明，多个属性以 , 隔开**

name属性：当请求中的属性名与方法参数名不一致时，如果不想传入失败，可以用name属性指定名称。(与value属性作用一致)。

UserController中部分代码，修改`query()`方法:

	@RequestMapping(value = "/user", method = RequestMethod.GET)
	public List<User> query(@RequestParam(name = "username") String nickname) {
		System.out.println(nickname);
		List<User> users = new ArrayList<User>();
		users.add(new User());
		users.add(new User());
		users.add(new User());
		return users;
	}

如方法中参数名为`nickname`，实际测试方法中传入参数名称为`username`

required属性：值为布尔类型，默认为true，当设置为false时，表示参数不是必须传入。

defaultValue属性：与required属性关联，当required为false时，可以为参数设置默认值。

UserController中部分代码，修改`query()`方法:

	@RequestMapping(value = "/user", method = RequestMethod.GET)
	public List<User> query(@RequestParam(name = "username", required = false, defaultValue = "tom") String nickname) {
		System.out.println(nickname);
		List<User> users = new ArrayList<User>();
		users.add(new User());
		users.add(new User());
		users.add(new User());
		return users;
	}

测试测试用例，可以不用携带参数，不会报错。测试用例部分代码（注释掉携带参数部分），修改`whenQuerySuccess()`方法：

	@Test
	public void whenQuerySuccess() throws Exception {
		mockMvc.perform(get("/user")
				//.param("username", "jojo")
				.contentType(MediaType.APPLICATION_JSON_UTF8))//请求的数据格式是json
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.length()").value(3));//期待返回集合的长度是3
	}

当查询参数很多时，可以封装参数对象。参数对象代码如下：

	package com.zhqx.dto;
	
	public class UserQueryCondition {
		
		private String username;
		
		private int age;
		
		private int ageTo;
		
		private String xxx;
	
		public String getUsername() {
			return username;
		}
	
		public void setUsername(String username) {
			this.username = username;
		}
	
		public int getAge() {
			return age;
		}
	
		public void setAge(int age) {
			this.age = age;
		}
	
		public int getAgeTo() {
			return ageTo;
		}
	
		public void setAgeTo(int ageTo) {
			this.ageTo = ageTo;
		}
	
		public String getXxx() {
			return xxx;
		}
	
		public void setXxx(String xxx) {
			this.xxx = xxx;
		}
	}

`UserController`类中，使用实体作为参数。修改`query()`方法，修改部分的代码如下：

	@RequestMapping(value = "/user", method = RequestMethod.GET)
	public List<User> query(UserQueryCondition condition) {
		//使用反射工具,打印实体信息
		System.out.println(ReflectionToStringBuilder.toString(condition, ToStringStyle.MULTI_LINE_STYLE));
			
		List<User> users = new ArrayList<User>();
		users.add(new User());
		users.add(new User());
		users.add(new User());
		return users;
	}

测试实例中，传递多个参数，自动封装到`UserQueryCondition`参数实体中。修改`whenQuerySuccess()`方法：

	@Test
	public void whenQuerySuccess() throws Exception {
		mockMvc.perform(get("/user")
				.param("username", "jojo")
				.param("age", "18")
				.param("ageTo", "20")
				.param("xxx", "yyy")
				.contentType(MediaType.APPLICATION_JSON_UTF8))//请求的数据格式是json
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.length()").value(3));//期待返回集合的长度是3
	}

> **使用Spring提供的分页对象Pageable，添加分页参数**

`import org.springframework.data.domain.Pageable;`修改`query()`方法，修改部分的代码如下：

	@RequestMapping(value = "/user", method = RequestMethod.GET)
	public List<User> query(UserQueryCondition condition, Pageable pageable) {
			
		System.out.println(ReflectionToStringBuilder.toString(condition, ToStringStyle.MULTI_LINE_STYLE));

		System.out.println(pageable.getPageSize());
		System.out.println(pageable.getPageNumber());
		System.out.println(pageable.getSort());

			
		List<User> users = new ArrayList<User>();
		users.add(new User());
		users.add(new User());
		users.add(new User());
		return users;
	}

在测试用例中，追加分页参数信息（部分代码）,修改`whenQuerySuccess()`方法：

	@Test
	public void whenQuerySuccess() throws Exception {
		mockMvc.perform(get("/user")
				.param("username", "jojo")
				.param("age", "18")
				.param("ageTo", "20")
				.param("xxx", "yyy")
				.param("size", "15")//每页显示15条数据
				.param("page", "3")//查询第三页数据
				.param("sort", "age,desc")//查询结果按照年龄降序排列
				.contentType(MediaType.APPLICATION_JSON_UTF8))//返回的数据格式是json
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.length()").value(3));//期待返回集合的长度是3
	}

运行测试用例，会打印出相关信息。

使用`@PageableDefault`注解指定分页的相关默认信息。修改`query()`方法，修改部分的代码如下：

	@RequestMapping(value = "/user", method = RequestMethod.GET)
	public List<User> query(UserQueryCondition condition, 
		@PageableDefault(page = 2, size = 17, sort = "username,asc") Pageable pageable) {
		
		System.out.println(ReflectionToStringBuilder.toString(condition, ToStringStyle.MULTI_LINE_STYLE));
		
		List<User> users = new ArrayList<User>();
		users.add(new User());
		users.add(new User());
		users.add(new User());
		return users;
	}

当没有相关参数时，会采用默认值。修改`whenQuerySuccess()`方法：这里由于size注释掉，会采用默认的size属性。

	@Test
	public void whenQuerySuccess() throws Exception {
		mockMvc.perform(get("/user")
				.param("username", "jojo")
				.param("age", "18")
				.param("ageTo", "20")
				.param("xxx", "yyy")
				//.param("size", "15")//每页显示15条数据
				.param("page", "3")//查询第三页数据
				.param("sort", "age,desc")//查询结果按照年龄降序排列
				.contentType(MediaType.APPLICATION_JSON_UTF8))//返回的数据格式是json
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.length()").value(3));//期待返回集合的长度是3
	}

### 二、增加测试用例-用户详情请求 ###
在`UserControllerTest`类中，增加测试用例：

	@Test
	public void whenGetInfoSuccess() throws Exception {
		mockMvc.perform(get("/user/1")
				.contentType(MediaType.APPLICATION_JSON_UTF8))//请求的数据格式是json
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.username").value("tom"));//期待返回结果中有username属性,值为tom
	}

在`UserController`类中，增加访问方法：

	@RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
	public User getInfo(@PathVariable String id) {
		User user = new User();
		user.setUsername("tom");
		return user;
	}

**@PathVariables注解中的属性与@RequestParam注解中的属性参数大体相同。所以为了避免麻烦，方法参数名与传递参数名保持一致**

方法参数名与表达式名不一致时，应该使用name属性指定。

表达式参数名为：`@RequestMapping(value = "/user/{id}", method = RequestMethod.GET)`，而参数名为`String idxxx`时

使用name属性指定一下，`(@PathVariable(name = "id") String idxxx)`。


增加测试用例，测试传递的参数值不符合规范的情况，`/user/1`是正常的访问路径，当`/user/a`时，理论上应该返回一个4开头的状态码。

	@Test
	public void whenGetInfoFail() throws Exception {
		mockMvc.perform(get("/user/a")
				.contentType(MediaType.APPLICATION_JSON_UTF8))
				.andExpect(status().isOk());
	}

这里我们执行测试用例时，测试是通过的。说明的返回的状态码是200

调整一下，期待返回的结果。

	@Test
	public void whenGetInfoFail() throws Exception {
		mockMvc.perform(get("/user/a")
				.contentType(MediaType.APPLICATION_JSON_UTF8))
				.andExpect(status().is4xxClientError());
	}

再次运行测试用例，测试结果并不通过，是因为，我们此时`UserController`中的方法，无法正确识别传递参数是否是数字。

只有能区别是否是数字时，才会有预期的4开头的返回状态码。

修改`UserController`中的方法，要求id参数只能是数字。

	@RequestMapping(value = "/user/{id:\\d+}", method = RequestMethod.GET)
	public User getInfo(@PathVariable String id) {
		User user = new User();
		user.setUsername("tom");
		return user;
	}

此时运行测试用例`whenGetInfoFail()`，测试通过，说明结果返回的是4开头的状态码。

使用`@JsonView`注解，区分不同请求的返回结果。例如，我们想在`public List<User> query(***)`方法中不返回`password`属性值，
而在`public User getInfo(***)`方法中返回`password`属性值。

使用`@JsonView`注解的步骤，1.使用接口来声明多个视图。2.在值对象的get方法上指定视图。3.在Controller方法上指定视图。

> **1.使用接口声明多个视图**

在User类中声明视图接口。

	package com.zhqx.dto;
	
	public class User {
		
		public interface UserSimpleView {};
		
		public interface UserDetailView extends UserSimpleView {};
		
		private String username;
		
		private String password;
	
		public String getUsername() {
			return username;
		}
	
		public void setUsername(String username) {
			this.username = username;
		}
	
		public String getPassword() {
			return password;
		}
	
		public void setPassword(String password) {
			this.password = password;
		}
		
	}

> **2.在值对象的get方法上指定视图**

	package com.zhqx.dto;
	
	import com.fasterxml.jackson.annotation.JsonView;
	
	public class User {
		
		public interface UserSimpleView {};
		
		public interface UserDetailView extends UserSimpleView {};
		
		private String username;
		
		private String password;
	
		@JsonView(UserSimpleView.class)
		public String getUsername() {
			return username;
		}
	
		public void setUsername(String username) {
			this.username = username;
		}
	
		@JsonView(UserDetailView.class)
		public String getPassword() {
			return password;
		}
	
		public void setPassword(String password) {
			this.password = password;
		}
		
	}

> **3.在Controller方法上指定视图**

修改UserController，在指定方法上使用注解添加视图。

	@RequestMapping(value = "/user", method = RequestMethod.GET)
	@JsonView(User.UserSimpleView.class)
	public List<User> query(UserQueryCondition condition,@PageableDefault(page = 2, size = 17, sort = "username,asc") Pageable pageable) {
		
		System.out.println(ReflectionToStringBuilder.toString(condition, ToStringStyle.MULTI_LINE_STYLE));
		
		System.out.println(pageable.getPageSize());
		System.out.println(pageable.getPageNumber());
		System.out.println(pageable.getSort());
		
		List<User> users = new ArrayList<User>();
		users.add(new User());
		users.add(new User());
		users.add(new User());
		return users;
	}
	
	@RequestMapping(value = "/user/{id:\\d+}", method = RequestMethod.GET)
	@JsonView(User.UserDetailView.class)
	public User getInfo(@PathVariable String id) {
		User user = new User();
		user.setUsername("tom");
		return user;
	}

修改测试用例，将返回的结果转换成字符串,以便更好观察结果。`whenQuerySuccess()`和`whenGetInfoSuccess()`方法

	@Test
	public void whenQuerySuccess() throws Exception {
		String result = mockMvc.perform(get("/user")
				.param("username", "jojo")
				.param("age", "18")
				.param("ageTo", "20")
				.param("xxx", "yyy")
				//.param("size", "15")//每页显示15条数据
				.param("page", "3")//查询第三页数据
				.param("sort", "age,desc")//查询结果按照年龄降序排列
				.contentType(MediaType.APPLICATION_JSON_UTF8))//返回的数据格式是json
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.length()").value(3))//期待返回集合的长度是3
				.andReturn().getResponse().getContentAsString();//将返回结果转成String
		
		System.out.println("Query:" + result);
	}
	
	@Test
	public void whenGetInfoSuccess() throws Exception {
		String result = mockMvc.perform(get("/user/1")
				.contentType(MediaType.APPLICATION_JSON_UTF8))//请求的数据格式是json
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.username").value("tom"))//期待返回结果中有username属性,值为tom
				.andReturn().getResponse().getContentAsString();//将返回结果转成String
		
		System.out.println("GetInfo:" + result);
	}


分别运行`whenQuerySuccess()`和`whenGetInfoSuccess()`测试用例。返回结果如下：

`Query:[{"username":null},{"username":null},{"username":null}]`

`GetInfo:{"username":"tom","password":null}`

从上，可以看出，符合我们预期的结果。Query方法查询没有返回password字段，GetInfo方法查询返回了password字段。

优化`UserController`类，精简代码。

	package com.zhqx.web.controller;
	
	import java.util.ArrayList;
	import java.util.List;
	
	import org.apache.commons.lang.builder.ReflectionToStringBuilder;
	import org.apache.commons.lang.builder.ToStringStyle;
	import org.springframework.data.domain.Pageable;
	import org.springframework.data.web.PageableDefault;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.RestController;
	
	import com.fasterxml.jackson.annotation.JsonView;
	import com.zhqx.dto.User;
	import com.zhqx.dto.UserQueryCondition;
	import com.zhqx.dto.User.UserSimpleView;
	
	@RestController
	@RequestMapping("/user")
	public class UserController {
		
		@GetMapping
		@JsonView(User.UserSimpleView.class)
		public List<User> query(UserQueryCondition condition,@PageableDefault(page = 2, size = 17, sort = "username,asc") Pageable pageable) {
			
			System.out.println(ReflectionToStringBuilder.toString(condition, ToStringStyle.MULTI_LINE_STYLE));
			
			System.out.println(pageable.getPageSize());
			System.out.println(pageable.getPageNumber());
			System.out.println(pageable.getSort());
			
			List<User> users = new ArrayList<User>();
			users.add(new User());
			users.add(new User());
			users.add(new User());
			return users;
		}
		
		@GetMapping("/{id:\\d+}")
		@JsonView(User.UserDetailView.class)
		public User getInfo(@PathVariable String id) {
			User user = new User();
			user.setUsername("tom");
			return user;
		}
	}

### 三、增加测试用例-用户创建请求 ###

在`UserControllerTest`类中增加测试用例：

	@Test
	public void whenCreateSuccess() throws Exception {
		String content =  "{\"username\":\"tom\",\"password\":null}";//构建的json字符串
		mockMvc.perform(post("/user")
				.contentType(MediaType.APPLICATION_JSON_UTF8)//请求的数据格式是json
				.content(content))//传输的数据内容
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.id").value("1"));//期待返回结果中有id属性,值为1
	}

测试运行测试用例，出现405提示，是因为此时我们的UserController类中是有/user对应的请求处理，只是不是POST方法。

修改User类，增加id属性，并配置返回视图。

	package com.zhqx.dto;
	
	import com.fasterxml.jackson.annotation.JsonView;
	
	public class User {
		
		public interface UserSimpleView {};
		
		public interface UserDetailView extends UserSimpleView {};
		
		private String id;
		
		private String username;
		
		private String password;
	
		@JsonView(UserSimpleView.class)
		public String getId() {
			return id;
		}
	
		public void setId(String id) {
			this.id = id;
		}
	
		@JsonView(UserSimpleView.class)
		public String getUsername() {
			return username;
		}
	
		public void setUsername(String username) {
			this.username = username;
		}
	
		@JsonView(UserDetailView.class)
		public String getPassword() {
			return password;
		}
	
		public void setPassword(String password) {
			this.password = password;
		}
		
	}

在UserController添加对应方法，处理POST请求的/user请求。

	@PostMapping
	public User create(User user) {

		System.out.println(user.getId());
		System.out.println(user.getUsername());
		System.out.println(user.getPassword());

		user.setId("1");
		return user;
	}

运行测试用例，测试通过，打印结果为：null,null,null。这里我们的User参数并没有自动封装我们构建的json参数。需要添加`@RequestBody`注解。

	@PostMapping
	public User create(@RequestBody User user) {

		System.out.println(user.getId());
		System.out.println(user.getUsername());
		System.out.println(user.getPassword());

		user.setId("1");
		return user;
	}

运行测试用例，打印结果为：null,tom,null。说明可以正常封装传递的参数。

> **日期类型处理，在Use类中添加Date类型参数。前后端分离时，建议在后台直接传递时间戳**

	package com.zhqx.dto;
	
	import java.util.Date;
	
	import com.fasterxml.jackson.annotation.JsonView;
	
	public class User {
		
		public interface UserSimpleView {};
		
		public interface UserDetailView extends UserSimpleView {};
		
		private String id;
		
		private String username;
		
		private String password;
		
		private Date birthday;
	
		@JsonView(UserSimpleView.class)
		public String getId() {
			return id;
		}
	
		public void setId(String id) {
			this.id = id;
		}
	
		@JsonView(UserSimpleView.class)
		public String getUsername() {
			return username;
		}
	
		public void setUsername(String username) {
			this.username = username;
		}
	
		@JsonView(UserDetailView.class)
		public String getPassword() {
			return password;
		}
	
		public void setPassword(String password) {
			this.password = password;
		}
	
		public Date getBirthday() {
			return birthday;
		}
	
		public void setBirthday(Date birthday) {
			this.birthday = birthday;
		}
		
	}

修改测试用例，添加日期参数，同时打印返回的结果。

	@Test
	public void whenCreateSuccess() throws Exception {
		Date date = new Date();
		System.out.println(date.getTime());
		String content = "{\"username\":\"tom\",\"password\":null,\"birthday\":"+date.getTime()+"}";//构建的json字符串
		String result = mockMvc.perform(post("/user")
				.contentType(MediaType.APPLICATION_JSON_UTF8)//请求的数据格式是json
				.content(content))//传输的数据内容
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.id").value("1"))//期待返回结果中有id属性,值为1
				.andReturn().getResponse().getContentAsString();//将返回结果转成String
		
		System.out.println("CreateSuccess:" + result);
	}

修改UserController。打印后台传递的时间参数。

	@PostMapping
	public User create(@RequestBody User user) {

		System.out.println(user.getId());
		System.out.println(user.getUsername());
		System.out.println(user.getPassword());
		System.out.println(user.getBirthday());

		user.setId("1");
		return user;
	}

运行测试用例，可以看到时间戳，在后台自动封装成Date日期，而后台传递到前台的Date日期，又会自动封装成时间戳。

**这里有个大大的问号，在低版本中，测试是正常的。但是在2.X的版本中，后台返回的结果并没有自动转化为时间戳。**

> **使用@Valid注解做参数验证**

当我们需要校验传递参数时，如果全部都自己判断，无疑会增加开发时间。这个时候我们可以使用注解来帮助我们做常规校验。

假设我们要求password属性不能为null。我们使用`@NotBlank`注解。

修改User类，在passoword属性前加`@NotBlank`需要引入：`import javax.validation.constraints.NotBlank;`。

	@NotBlank
	private String password;

此时我们运行测试用例，发现测试仍然可以正常通过，是因为我们还需要在`UserController`中添加`@Valid`注解。

	@PostMapping
	public User create(@Valid @RequestBody User user) {

		System.out.println(user.getId());
		System.out.println(user.getUsername());
		System.out.println(user.getPassword());
		System.out.println(user.getBirthday());

		user.setId("1");
		return user;
	}

此时，运行测试用例，会显示测试不通过，状态码为400。

在使用@Valid注解时，我们会发现，并没有进入`create()`方法中，在有些时候，我们是需要进入方法做记录的。

我们使用`BindingResult`类来处理。

	@PostMapping
	public User create(@Valid @RequestBody User user, BindingResult errors) {
		
		if (errors.hasErrors()) {
			errors.getAllErrors().stream().forEach(error -> System.out.println(error.getDefaultMessage()));
		}

		System.out.println(user.getId());
		System.out.println(user.getUsername());
		System.out.println(user.getPassword());
		System.out.println(user.getBirthday());

		user.setId("1");
		return user;
	}

此时再次运行测试用例，会发现测试通过。控制台会打印出错误信息：`must not be blank`。

### 四、增加测试用例-用户信息修改和删除 ###

> **用户信息修改：**

我们在User类中增加一个校验，要求传递的日期必须是过去的时间。需要导入：`import javax.validation.constraints.Past;`

	@Past
	private Date birthday;

增加用户信息修改测试用例：

	@Test
	public void whenUpdateSuccess() throws Exception {
		//构建一个过去的时间
		Date date = new Date(LocalDateTime.now().plusYears(1).atZone(ZoneId.systemDefault()).toInstant().toEpochMilli());
		System.out.println(date.getTime());
		String content = "{\"id\":\"1\", \"username\":\"tom\",\"password\":null,\"birthday\":" + date.getTime() + "}";
		String reuslt = mockMvc.perform(put("/user/1")
				.contentType(MediaType.APPLICATION_JSON_UTF8)//请求的数据格式是json
				.content(content))//传输的数据内容
				.andExpect(status().isOk())//期待返回的状态码是200(即正常)
				.andExpect(jsonPath("$.id").value("1"))//期待返回结果中有id属性,值为1
				.andReturn().getResponse().getContentAsString();//将返回结果转成String
		
		System.out.println(reuslt);
	}

增加对应请求，详细打印每个错误的验证信息：

	@PutMapping("/{id:\\d+}")
	public User update(@Valid @RequestBody User user, BindingResult errors) {
		
		if (errors.hasErrors()) {
			errors.getAllErrors().stream().forEach(error -> {
				FieldError fieldError = (FieldError) error;
				String message = fieldError.getField() + " " + error.getDefaultMessage();
				System.out.println(message);
			}
			);
		}

		System.out.println(user.getId());
		System.out.println(user.getUsername());
		System.out.println(user.getPassword());
		System.out.println(user.getBirthday());

		user.setId("1");
		return user;
	}

运行测试用例，控制台打印出错误信息：`birthday must be a past date`和`password must not be blank`。

更改默认的校验信息，在User类中，在校验注解上，使用message属性：

	@NotBlank(message = "密码不能为空")
	private String password;
	
	@Past(message = "生日必须是过去的时间")
	private Date birthday;

> **自定义校验注解校验字段**

增加一个HelloService并实现，主要用来说明，在注解校验器中可以引入项目中的类帮助我们完成校验逻辑。

	package com.zhqx.service;
	
	public interface HelloService {
		String greeting(String name);
	}

实现类：

	package com.zhqx.service.impl;
	
	import org.springframework.stereotype.Service;
	import com.zhqx.service.HelloService;
	
	@Service
	public class HelloServiceImpl implements HelloService {
	
		@Override
		public String greeting(String name) {
			System.out.println("greeting");
			return "hello "+name;
		}
		
	}

创建自定义注解：

注解类：

	package com.zhqx.validator;
	
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;
	
	import javax.validation.Constraint;
	import javax.validation.Payload;
	
	@Target({ElementType.METHOD, ElementType.FIELD})//注解使用位置,可以在方法上、可以在属性上
	@Retention(RetentionPolicy.RUNTIME)//运行时注解
	@Constraint(validatedBy = MyConstraintValidator.class)//当前注解需要使用哪个逻辑类处理
	public @interface MyConstraint {
		
		//自定义注解默认需要写的三个属性
		
		String message();
	
		Class<?>[] groups() default { };
	
		Class<? extends Payload>[] payload() default { };
	
	}

注解相应的校验器类：

	package com.zhqx.validator;
	
	import javax.validation.ConstraintValidator;
	import javax.validation.ConstraintValidatorContext;
	
	import org.springframework.beans.factory.annotation.Autowired;
	
	import com.zhqx.service.HelloService;
	
	//参数1,需要验证的注解,参数2,验证的参数类型、
	public class MyConstraintValidator implements ConstraintValidator<MyConstraint, Object> {
		
		//在校验器中可以注入项目中的任意类来帮助实现校验逻辑
		//这里不需要使用@Component注解,实现ConstraintValidator接口，spring会自动为我们添加
		@Autowired
		private HelloService helloService;
		
		//校验初始化
		@Override
		public void initialize(MyConstraint constraintAnnotation) {
			System.out.println("my validator init");
		}
	
		//校验逻辑
		@Override
		public boolean isValid(Object value, ConstraintValidatorContext context) {
			helloService.greeting("tom");
			System.out.println(value);
			
			//当返回false时，表示校验失败，可以看到校验信息。返回true，则表示校验通过
			return false;
		}
	
	}

使用自定义注解，在User类中的username属性上使用注解。

	@MyConstraint(message = "这是一个测试")
	private String username;

运行测试用例`whenUpdateSuccess()`。我们的校验逻辑返回的永远是false，所以我们可以在控制台看到校验错误信息：`username 这是一个测试`。

> **用户信息删除**

在UserControllerTest类中增加用户信息删除测试用例：

	@Test
	public void whenDeleteSuccess() throws Exception {
		mockMvc.perform(delete("/user/1")
				.contentType(MediaType.APPLICATION_JSON_UTF8))
				.andExpect(status().isOk());
	}

在UserController类中增加用户信息删除的对应请求：

	@DeleteMapping("/{id:\\d+}")
	public void delete(@PathVariable String id) {
		System.out.println(id);
	}

运行测试用例，显示测试通过。

### 五、服务异常处理 ###
Springboot默认的错误处理控制器是`BasicErrorController`。在默认的处理器中，Springboot针对浏览器和客户端发出的错误请求分别做了处理。

默认的处理错误的请求是`/error`。
	
	//如果请求头中的参数有text/html，则执行此方法。返回的是一个html页面
	@RequestMapping(produces = "text/html")
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}

	//如果请求头中的参数没有text/html，则执行此方法。返回的是一个json格式数据
	@RequestMapping
	@ResponseBody
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<>(body, status);
	}

**这里在同一个访问请求下如何返回不同的结果。Springboot做了处理，后面如果有需要可以参考。**

> **1.Springboot处理错误请求**

**需要启动Springboot服务。**

修改UserController类中的POST请求/user（用户创建），将BindingResult errors参数删除，根据前面的测试，我们知道。当请求校验错误时，

是没有办法进入方法体的。我们分别通过浏览器和利用客户端发送请求观察结果。

	@PostMapping
	public User create(@Valid @RequestBody User user) {

		System.out.println(user.getId());
		System.out.println(user.getUsername());
		System.out.println(user.getPassword());
		System.out.println(user.getBirthday());

		user.setId("1");
		return user;
	}

我通过客户端发送`http://localhost:8080/user`请求（POST请求），可以返回400状态码，返回内容如下：

	{
	"timestamp": "2020-08-07T06:53:26.451+0000",
	"status": 400,
	"error": "Bad Request",
	"errors":[
	{
	"codes":["NotBlank.user.password", "NotBlank.password", "NotBlank.java.lang.String", "NotBlank"],
	"arguments":[{"codes":["user.password", "password" ], "arguments": null, "defaultMessage": "password",…],
	"defaultMessage": "密码不能为空",
	"objectName": "user",
	"field": "password",
	"rejectedValue": null,
	"bindingFailure": false,
	"code": "NotBlank"
	},
	{
	"codes":["MyConstraint.user.username", "MyConstraint.username", "MyConstraint.java.lang.String", "MyConstraint"],
	"arguments":[{"codes":["user.username", "username" ], "arguments": null, "defaultMessage": "username",…],
	"defaultMessage": "这是一个测试",
	"objectName": "user",
	"field": "username",
	"rejectedValue": null,
	"bindingFailure": false,
	"code": "MyConstraint"
	}
	],
	"message": "Validation failed for object='user'. Error count: 2",
	"path": "/user"
	}

这里我们可以看到，尽管没有进入方法体，Springboot会将详细的验证信息返回。这里Springboot对400错误状态码，做了默认处理。

这里我们再修改/user/{id:\\d+}（get请求）。在方法体中(`getInfo()方法`)我们抛出一个运行期异常，看看对状态码500的处理。

	@GetMapping("/{id:\\d+}")
	@JsonView(User.UserDetailView.class)
	public User getInfo(@PathVariable String id) {
		
		throw new RuntimeException("user not exist");
		
		//User user = new User();
		//user.setUsername("tom");
		//return user;
	}

我们通过浏览器和客户端分别访问`http://localhost:8080/user/1`，浏览器的返回的是一个html页面，包含如下内容：

	Whitelabel Error Page
	This application has no explicit mapping for /error, so you are seeing this as a fallback.
	
	Fri Aug 07 15:04:37 CST 2020
	There was an unexpected error (type=Internal Server Error, status=500).
	user not exist

客户端返回的是一个json信息，结果如下：
	
	{
	"timestamp": "2020-08-07T07:05:04.897+0000",
	"status": 500,
	"error": "Internal Server Error",
	"message": "user not exist",
	"path": "/user/1"
	}

当我们在浏览器中访问一个不存在的请求时（404状态码），返回的是一个html，包含如下内容：

	Whitelabel Error Page
	This application has no explicit mapping for /error, so you are seeing this as a fallback.
	
	Fri Aug 07 15:07:58 CST 2020
	There was an unexpected error (type=Not Found, status=404).
	No message available

> **2.如何自定义异常处理页面（web浏览器访问）**

在`src/main/resources`目录下创建目录`resources`，在`resources`目录下创建`error`目录。在`error`目录下创建自定义的`404.html`。

	<!DOCTYPE html>
	<html>
	<head>
	<meta charset="UTF-8">
	<title>404</title>
	</head>
	<body>
		您访问的页面不存在!
	</body>
	</html>

此时再次通过浏览器访问不存在的请求时，则返回的是我们自定义的html页面。客户端的请求不受影响。

同样的道理，我们可以自定义500状态的返回页面。在`error`目录下创建`500.html`

	<!DOCTYPE html>
	<html>
	<head>
	<meta charset="UTF-8">
	<title>500</title>
	</head>
	<body>
		服务器内部错误!
	</body>
	</html>

此时通过浏览器访问`http://localhost:8080/user/1`我们会发现返回的是自定义的500错误处理页面。客户端的请求不受影响。

> **3.如何自定义异常处理json内容（客户端访问）**

上面是自定义浏览器中发送的请求出现错误时，会出现的错误页面。那么如何自定义通过客户端访问请求时，出现错误时返回的json内容？

构造自定义异常`UserNotExistException`（用户不存在时出现的异常）。

	package com.zhqx.exception;
	
	public class UserNotExistException extends RuntimeException {
		
		/**
		 * 
		 */
		private static final long serialVersionUID = 1L;
		private String id;
		
		public UserNotExistException(String id) {
			super("user not exist...");
			this.id = id;
		}
	
		public String getId() {
			return id;
		}
	
		public void setId(String id) {
			this.id = id;
		}
	
	}

修改`UserController`类中，修改`/user/{id:\\d+}`（get请求）。在方法体中(`getInfo()方法`)，将抛出异常改为我们自定义异常。

	@GetMapping("/{id:\\d+}")
	@JsonView(User.UserDetailView.class)
	public User getInfo(@PathVariable String id) {
		
		throw new UserNotExistException(id);
		
		//User user = new User();
		//user.setUsername("tom");
		//return user;
	}

通过客户端访问`http://localhost:8080/user/1`。我们会看到如下返回结果:

	{
	"timestamp": "2020-08-07T07:39:02.677+0000",
	"status": 500,
	"error": "Internal Server Error",
	"message": "user not exist...",
	"path": "/user/1"
	}

这里我们可以看到返回信息变成可我们自定义异常错误信息。如何将我们希望返回的id值显示到错误处理的json内容中呢？

因为目前为止，无论是自定义的错误异常处理，还是默认的错误异常处理，都是Springboot默认帮我们处理的。所以我们需要自定义异常处理控制器
来处理自定义异常。

新建异常处理控制器，`ControllerExceptionHandler`。

	package com.zhqx.web.controller;
	
	import java.util.HashMap;
	import java.util.Map;
	
	import org.springframework.http.HttpStatus;
	import org.springframework.web.bind.annotation.ControllerAdvice;
	import org.springframework.web.bind.annotation.ExceptionHandler;
	import org.springframework.web.bind.annotation.ResponseBody;
	import org.springframework.web.bind.annotation.ResponseStatus;
	
	import com.zhqx.exception.UserNotExistException;
	
	@ControllerAdvice
	public class ControllerExceptionHandler {
	
		//当出现UserNotExistException时,使用该方法处理异常
		@ExceptionHandler(UserNotExistException.class)
		@ResponseBody//返回json格式内容
		@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)//返回的HTTP状态码
		public Map<String, Object> handleUserNotExistException(UserNotExistException ex) {
			Map<String, Object> result = new HashMap<>();
			result.put("id", ex.getId());
			result.put("message", ex.getMessage());
			return result;
		}
	
	}

再次通过客户端访问`http://localhost:8080/user/1`。我们会看到如下返回结果:

	{
	"id": "1",
	"message": "user not exist..."
	}

这里我们成功的将返回信息，自定义成我们需要的返回信息了。

### 六、RESTful API拦截处理 ###
在Spring中，有3种拦截机制，可以帮助我们拦截请求。分别是过滤器（Filter）、拦截器（Interceptor）、切片（Aspect）。

> **1.记录所用服务处理时间，使用过滤器（Filter）来完成。**

新建`TimeFilter`类，该类需要实现`javax.servlet.Filter`接口。

	package com.zhqx.web.filter;
	
	import java.io.IOException;
	import java.util.Date;
	
	import javax.servlet.Filter;
	import javax.servlet.FilterChain;
	import javax.servlet.FilterConfig;
	import javax.servlet.ServletException;
	import javax.servlet.ServletRequest;
	import javax.servlet.ServletResponse;
	
	import org.springframework.stereotype.Component;
	
	//使用该注解，将组件交给Spring管理
	@Component
	public class TimeFilter implements Filter {
	
		@Override
		public void init(FilterConfig filterConfig) throws ServletException {
			// TODO Auto-generated method stub
			System.out.println("time filter init");
		}
	
		@Override
		public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
				throws IOException, ServletException {
			// TODO Auto-generated method stub
			System.out.println("time filter start");
			long start = new Date().getTime();
			chain.doFilter(request, response);
			System.out.println("time filter 耗时:" + (new Date().getTime() - start));
			System.out.println("time filter finish");
		}
	
		@Override
		public void destroy() {
			// TODO Auto-generated method stub
			System.out.println("time filter destroy");
		}
	
	}

修改`UserController`类中的`getInfo()`方法，方便观察过滤器处理过程。

	@GetMapping("/{id:\\d+}")
	@JsonView(User.UserDetailView.class)
	public User getInfo(@PathVariable String id) {
		//throw new UserNotExistException(id);
		System.out.println("进入getInfo方法");
		User user = new User();
		user.setUsername("tom");
		return user;
	}

启动服务，访问getInfo服务。观察控制台打印信息。（访问其他服务同样会触发过滤器）。

> **2.使用第三方框架实现好的过滤器。**

实际开发过程中，可能会用到第三方提供的过滤器，但是第三方过滤器，通常都是封装好的，我们没办法使用`@Component`注解添加过滤器。

在传统的SpringMVC中，可以使用xml配置文件来配置过滤器，引入到项目。但是Springboot中没有xml配置文件，我们该如何实现。

修改TimeFilter过滤器，将`@Component`注解注释掉。修改如下：

	//使用该注解，将组件交给Spring管理
	//@Component
	public class TimeFilter implements Filter {
		......

自定义配置类`WebConfig`，添加第三方框架提供的过滤器。

	import java.util.ArrayList;
	import java.util.List;
	
	import org.springframework.boot.web.servlet.FilterRegistrationBean;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	
	import com.zhqx.web.filter.TimeFilter;
	
	@Configuration
	public class WebConfig {
		
		@Bean
		public FilterRegistrationBean<TimeFilter> timeFilter() {
			
			FilterRegistrationBean<TimeFilter> registrationBean = new FilterRegistrationBean<TimeFilter>();
			
			TimeFilter timeFilter = new TimeFilter();
			registrationBean.setFilter(timeFilter);
			
			//定义需要过滤器验证的请求地址
			List<String> urls = new ArrayList<>();
			//所有请求都生效,实际中可以自定义
			urls.add("/*");
			registrationBean.setUrlPatterns(urls);
			
			return registrationBean;
			
		}
	}

启动服务，访问getInfo服务，同样发现过滤器生效了。

在Springboot中使用过滤器来拦截请求，我们只能拿到HTTP的请求（Request）和响应（Response）。我们能从请求和响应中获得一些参数。

我们并不能知道当前发送过来的请求是由哪个控制器的哪个方法来处理的。因为我们实现的TimeFilter是实现Filter接口的。

Filter接口是J2EE的规范，它并不了解与Spring相关的内容。我们可是使用拦截器，帮助我们进一步获取想要的信息。

> **3.记录服务所用时间，同时获取调用的是哪个控制器和哪个方法，使用拦截器（Interceptor）实现**

创建拦截器`TimeInterceptor`

	import java.util.Date;
	
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	
	import org.springframework.stereotype.Component;
	import org.springframework.web.method.HandlerMethod;
	import org.springframework.web.servlet.HandlerInterceptor;
	import org.springframework.web.servlet.ModelAndView;
	
	@Component
	public class TimeInterceptor implements HandlerInterceptor {
	
		// 访问某个控制器(controller)的某个方法被调用之前,该方法会被调用
		@Override
		public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
				throws Exception {
			System.out.println("preHandle");
			// 打印控制器(Controller)的类名
			System.out.println(((HandlerMethod) handler).getBean().getClass().getName());
			// 打印控制器(Controller)的方法名
			System.out.println(((HandlerMethod) handler).getMethod().getName());
			request.setAttribute("startTime", new Date().getTime());
			return false;
		}
	
		// 方法被调用之后,该方法会被调用，如果方法中出现异常，则不会调用
		@Override
		public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
				ModelAndView modelAndView) throws Exception {
			System.out.println("postHandle");
			Long start = (Long) request.getAttribute("startTime");
			System.out.println("time interceptor 耗时:" + (new Date().getTime() - start));
		}
	
		// 无论服务是否抛出异常,该方法都会被调用,当出现异常时,参数Exception ex会有值
		@Override
		public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
				throws Exception {
			System.out.println("afterCompletion");
			Long start = (Long) request.getAttribute("startTime");
			System.out.println("time interceptor 耗时:" + (new Date().getTime() - start));
			System.out.println("ex is " + ex);
		}
	
	}

与过滤器不相同的时，定义拦截器之后，如果想让它生效，仅仅使用`@Component`注解是不起作用的。还需要在配置类中，注册该拦截器。

修改WebConfig类

	import java.util.ArrayList;
	import java.util.List;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.web.servlet.FilterRegistrationBean;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
	import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
	
	import com.zhqx.web.filter.TimeFilter;
	import com.zhqx.web.interceptor.TimeInterceptor;
	
	@Configuration
	public class WebConfig implements WebMvcConfigurer {
		
		@Autowired
		private TimeInterceptor timeInterceptor;
		
		
		//注册自定义拦截器
		@Override
		public void addInterceptors(InterceptorRegistry registry) {
			registry.addInterceptor(timeInterceptor);
		}
	
		@Bean
		public FilterRegistrationBean<TimeFilter> timeFilter() {
			
			FilterRegistrationBean<TimeFilter> registrationBean = new FilterRegistrationBean<TimeFilter>();
			
			TimeFilter timeFilter = new TimeFilter();
			registrationBean.setFilter(timeFilter);
			
			//定义需要过滤器验证的请求地址
			List<String> urls = new ArrayList<>();
			//所有请求都生效,实际中可以自定义
			urls.add("/*");
			registrationBean.setUrlPatterns(urls);
			
			return registrationBean;
			
		}
	}

启动服务器，访问getInfo服务，发现控制台打印信息如下：

	time filter start
	preHandle
	com.zhqx.web.controller.UserController
	getInfo
	进入getInfo方法
	postHandle
	time interceptor 耗时:68
	afterCompletion
	time interceptor 耗时:68
	ex is null
	time filter 耗时:77
	time filter finish

修改`UserController`类的`getInfo`方法，如果方法抛出异常会怎样。

	@GetMapping("/{id:\\d+}")
	@JsonView(User.UserDetailView.class)
	public User getInfo(@PathVariable String id) {
		throw new UserNotExistException(id);
		//System.out.println("进入getInfo方法");
		//User user = new User();
		//user.setUsername("tom");
		//return user;
	}

启动服务，访问getInfo方法，发现控制台打印信息如下，`TimeInterceptor`拦截器中的`postHandle`方法没有执行。

	time filter start
	preHandle
	com.zhqx.web.controller.UserController
	getInfo
	进入getInfo方法
	afterCompletion
	time interceptor 耗时:47
	ex is null
	time filter 耗时:57
	time filter finish

此时，我们发现，出现异常时，正常打印的异常信息仍然时null，这是因为，我们抛出的异常是UserNotExistException，这个异常是我们自定义的。

并且，我们已经通过自定义异常处理控制器（`ControllerExceptionHandler`类），来处理了这个异常。

而使用`@ControllerAdvice`注解的`ControllerExceptionHandler`是在`TimeInterceptor`拦截器的`afterCompletion`方法之前处理异常的。

所以在`afterCompletion`方法中，拿不到异常，打印结果为null。如果我们将抛出的异常改为默认没做处理的异常。则会看到打印的异常。

修改`UserController`类的`getInfo`方法：

	@GetMapping("/{id:\\d+}")
	@JsonView(User.UserDetailView.class)
	public User getInfo(@PathVariable String id) {
		System.out.println("进入getInfo方法");
		throw new RuntimeException("user not exist");
		//throw new UserNotExistException(id);
		//User user = new User();
		//user.setUsername("tom");
		//return user;
	}

启动服务，再次访问getInfo方法，打印结果如下：

	time filter start
	preHandle
	com.zhqx.web.controller.UserController
	getInfo
	进入getInfo方法
	afterCompletion
	time interceptor 耗时:16
	ex is java.lang.RuntimeException: user not exist
	
	......省略错误信息
	
	preHandle
	org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController
	error
	postHandle
	time interceptor 耗时:50
	afterCompletion
	time interceptor 耗时:51
	ex is null

**这里我们可以看到`TimeInteceptor`中的`afterCompletion`方法能够正常打印出异常信息了。而后面，我们发现Springboot默认的错误处理控制器同样被拦截器拦截了。拦截器会拦截所有关联的控制器。**

> **4.获取方法实际的参数值，使用切片完成（Aspect）实现**

使用Spring AOP简介。切片（类）要点：

切入点（注解）：1.在哪些方法上起作用。2.在什么时候起作用

增强（方法）：起作用时执行的业务逻辑。

使用切片时，需要使用`@Aspect`注解，需要在pom文件中引入aop相关的jar包。(`zhqx-security-demo`项目的pom文件中)。

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-aop</artifactId>
	</dependency>

定义一个切片类。`TimeAspect`。

	import java.util.Date;
	
	import org.aspectj.lang.ProceedingJoinPoint;
	import org.aspectj.lang.annotation.Around;
	import org.aspectj.lang.annotation.Aspect;
	import org.springframework.stereotype.Component;
	
	@Aspect
	@Component
	public class TimeAspect {
		
	/*
	 * 	注解是包含了@Before注解（方法调用之前）。@After注解（方法成功返回之后）。@AfterThrowing（方法抛出异常时，方法会调用）
	 * 	@Around是一种综合的用法，涵盖@Before、@After、@AfterThrowing，可以用它处理3个注解中的任何一个。
	 * 	execution([任何的返回值都可以]  com.imooc.web.controller.UserController.[任何一个方法]([包含任何参数]))
	 * 	Spring官方文档面些切面编程提示了用法。
	 * 	ProceedingJoinPoint 包含当前拦截的方法的一些信息
	 */
		@Around("execution(* com.zhqx.web.controller.UserController.*(..))")
		public Object handleControllerMethod(ProceedingJoinPoint pjp) throws Throwable {
			
			System.out.println("time aspect start");
			
			Object[] args = pjp.getArgs();
			for (Object arg : args) {
				System.out.println("arg is " + arg);
			}
			
			long start = new Date().getTime();
			//调用后续方法
			Object object = pjp.proceed();
			
			System.out.println("time aspect 耗时:" + (new Date().getTime() - start));
			
			System.out.println("time aspect end");
			
			return object;
		}
	
	}

将上面修改的getInfo方法恢复：

	@GetMapping("/{id:\\d+}")
	@JsonView(User.UserDetailView.class)
	public User getInfo(@PathVariable String id) {
		System.out.println("进入getInfo方法");
		//throw new RuntimeException("user not exist");
		//throw new UserNotExistException(id);
		User user = new User();
		user.setUsername("tom");
		return user;
	}

启动服务，再次访问getInfo方法，`http://localhost:8080/user/1`,观察控制台打印信息。

	time filter start
	preHandle
	com.zhqx.web.controller.UserController$$EnhancerBySpringCGLIB$$36aca369
	getInfo
	time aspect start
	arg is 1
	进入getInfo方法
	time aspect 耗时:3
	time aspect end
	postHandle
	time interceptor 耗时:76
	afterCompletion
	time interceptor 耗时:77
	ex is null
	time filter 耗时:85
	time filter finish

这里我们看到打印的参数为1。总结一下RESTFul API的拦截顺序是：

`Filter -> Interceptor -> ControllerAdvice -> Aspect -> Controller`

### 七、Springboot文件上传和下载 ###

在`TestUserController`中增加新的测试方法：

	@Test
	public void whenUploadSuccess() throws Exception {
		String result = mockMvc.perform(multipart("/file")
				.file(new MockMultipartFile("file", "test.txt", "multipart/form-data", "hello upload".getBytes("UTF-8"))))
				.andExpect(status().isOk())
				.andReturn().getResponse().getContentAsString();
		System.out.println(result);
	}

增加一个文件对象`FileInfo`：

	package com.zhqx.dto;
	
	public class FileInfo {
		
		public FileInfo(String path){
			this.path = path;
		}
		
		private String path;
	
		public String getPath() {
			return path;
		}
	
		public void setPath(String path) {
			this.path = path;
		}
		
	}

在新增控制器之前，我们需要引入`org.apache.commons.io.IOUtils`;（`zhqx-security-demo`项目的pom文件中）。

	<dependency>
		<groupId>commons-io</groupId>
		<artifactId>commons-io</artifactId>
	</dependency>

增加一个新的控制器`FileController`、用来处理文件上传和下载。

	package com.zhqx.web.controller;
	
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.InputStream;
	import java.io.OutputStream;
	import java.util.Date;
	
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	
	import org.apache.commons.io.IOUtils;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.PostMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	import org.springframework.web.multipart.MultipartFile;
	
	import com.zhqx.dto.FileInfo;
	
	@RestController
	@RequestMapping("/file")
	public class FileController {
	
		private String folder = "D://";
	
		@PostMapping
		public FileInfo upload(MultipartFile file) throws Exception {
	
			System.out.println(file.getName());
			System.out.println(file.getOriginalFilename());
			System.out.println(file.getSize());
	
			File localFile = new File(folder, new Date().getTime() + ".txt");
	
			file.transferTo(localFile);
	
			return new FileInfo(localFile.getAbsolutePath());
		}
	
		@GetMapping("/{id}")
		public void download(@PathVariable String id, HttpServletRequest request, HttpServletResponse response) throws Exception {
	
			try (InputStream inputStream = new FileInputStream(new File(folder, id + ".txt"));
					OutputStream outputStream = response.getOutputStream();) {
				
				response.setContentType("application/x-download");
				response.addHeader("Content-Disposition", "attachment;filename=test.txt");
				
				IOUtils.copy(inputStream, outputStream);
				outputStream.flush();
			} 
	
		}
	
	}

运行测试方法`whenUploadSuccess()`、测试通过，会在D盘根目录生成一个类似文件：`1597042497676.txt`。

通过浏览器访问：`http://localhost:8080/file/1597042497676`则会正常下载txt文件。

### 八、多线程提高REST服务性能 ###

> **正常http请求过程**

新建控制器`AsyncController`

	package com.zhqx.web.async;
	
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	@RestController
	public class AsyncController {
		
		private Logger logger = LoggerFactory.getLogger(getClass());
		
		@RequestMapping("/order")
		public String order() throws Exception {
			logger.info("主线程开始");
			/*这里沉睡1秒,表示处理下单逻辑*/
			Thread.sleep(1000);
			logger.info("主线程返回");
			return "success";
		}
	}

这种是标准的同步处理，使用浏览器访问请求：`http://localhost:8080/order`,返回结果包含如下内容：

	15:33:40.636  INFO 19404 --- [nio-8080-exec-1] com.zhqx.web.async.AsyncController       : 主线程开始
	15:33:41.636  INFO 19404 --- [nio-8080-exec-1] com.zhqx.web.async.AsyncController       : 主线程返回

可以得知处理的线程是`nio-8080-exec-1`，执行时间相差1秒，这就是正常情况下，同步的处理方式。

> **使用Callable异步处理http请求过程**

修改`order()`方法

	@RequestMapping("/order")
	public Callable<String> order() throws Exception {
		logger.info("主线程开始");
		Callable<String> result = new Callable<String>() {
			@Override
			public String call() throws Exception {
				logger.info("副线程开始");
				/*这里沉睡1秒,表示处理下单逻辑*/
				Thread.sleep(1000);
				logger.info("副线程返回");
				return "success";
			}
		};
		logger.info("主线程返回");
		return result;
	}

为了更好的 观察结果，将前面所引入的过滤器、拦截器停止引入。（`WebConfig`类中注释相应代码，以及`TimeInterceptor`类中注释相应代码）。

启动服务，通过浏览器访问：`http://localhost:8080/order`,（访问2次）返回结果包含如下内容：

	16:00:41.668  INFO 22212 --- [nio-8080-exec-1] com.zhqx.web.async.AsyncController       : 主线程开始
	16:00:41.669  INFO 22212 --- [nio-8080-exec-1] com.zhqx.web.async.AsyncController       : 主线程返回
	16:00:41.672  WARN 22212 --- [nio-8080-exec-1] o.s.w.c.request.async.WebAsyncManager    : 
	!!!
	An Executor is required to handle java.util.concurrent.Callable return values.
	Please, configure a TaskExecutor in the MVC config under "async support".
	The SimpleAsyncTaskExecutor currently in use is not suitable under load.
	-------------------------------
	Request URI: '/order'
	!!!
	16:00:41.677  INFO 22212 --- [      MvcAsync1] com.zhqx.web.async.AsyncController       : 副线程开始
	16:00:42.678  INFO 22212 --- [      MvcAsync1] com.zhqx.web.async.AsyncController       : 副线程返回
	16:00:46.633  INFO 22212 --- [nio-8080-exec-4] com.zhqx.web.async.AsyncController       : 主线程开始
	16:00:46.633  INFO 22212 --- [nio-8080-exec-4] com.zhqx.web.async.AsyncController       : 主线程返回
	16:00:46.633  INFO 22212 --- [      MvcAsync2] com.zhqx.web.async.AsyncController       : 副线程开始
	16:00:47.635  INFO 22212 --- [      MvcAsync2] com.zhqx.web.async.AsyncController       : 副线程返回

这里我们可以看到，我们的主线程在处理请求时，没有经过停顿就返回了，请求都是开启副线程来处理。

这里的提示信息表示的是，我们需要使用自己的线程管理类来代替`Springboot`默认的线程管理。Springboot默认提供的线程管理，是不重用线程的。

> **更加复杂的场景，使用`DeferredResult`**

假设接受下单请求的应用和处理下单请求的应用是2台服务器。当有下单请求时，下单服务器会起一个线程1，发送下单消息到消息队列中。

处理订单服务器会监听消息队列，当发现有下单消息时，会运行处理逻辑。处理完成后，将处理结果再放回到消息队列。

与此同时，在下单服务器，会有一个线程2也在监听消息队列，当监听到处理完毕的消息时，返回http响应结果。

**模拟定义消息队列：**
	
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.stereotype.Component;
	
	/*代表消息队列*/
	@Component
	public class MockQueue {
		
		private Logger logger = LoggerFactory.getLogger(getClass());
	
		/*下单消息*/
		private String placeOrder;
		/*下单完成消息*/
		private String completeOrder;
	
		public String getPlaceOrder() {
			return placeOrder;
		}
	
		public void setPlaceOrder(String placeOrder) throws Exception {
			//下单服务器启动一个线程来处理下单请求
			new Thread(() -> {
				logger.info("接到下单请求, " + placeOrder);
				try {
					/*模拟下单过程*/
					Thread.sleep(1000);
				} catch (Exception e) {
					e.printStackTrace();
				}
				this.completeOrder = placeOrder;
				logger.info("下单请求处理完毕," + placeOrder);
			}).start();
		}
	
		public String getCompleteOrder() {
			return completeOrder;
		}
	
		public void setCompleteOrder(String completeOrder) {
			this.completeOrder = completeOrder;
		}
	
	}

**模拟定义订单处理服务器：**

	package com.zhqx.web.async;
	
	import java.util.HashMap;
	import java.util.Map;
	
	import org.springframework.stereotype.Component;
	import org.springframework.web.context.request.async.DeferredResult;
	/*代表处理服务器*/
	@Component
	public class DeferredResultHolder {
		
		/*String:代表订单号;DeferredResult<String>:代表处理结果*/
		private Map<String, DeferredResult<String>> map = new HashMap<String, DeferredResult<String>>();
	
		public Map<String, DeferredResult<String>> getMap() {
			return map;
		}
	
		public void setMap(Map<String, DeferredResult<String>> map) {
			this.map = map;
		}
		
	}

**模拟下单过程-线程，修改`AsyncController`类的`order()`方法**

	package com.zhqx.web.async;
	
	import org.apache.commons.lang.RandomStringUtils;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	import org.springframework.web.context.request.async.DeferredResult;
	
	@RestController
	public class AsyncController {
		
		private Logger logger = LoggerFactory.getLogger(getClass());
		
		@Autowired
		private MockQueue mockQueue;
		
		@Autowired
		private DeferredResultHolder deferredResultHolder;
		
		@RequestMapping("/order")
		public DeferredResult<String> order() throws Exception {
			logger.info("主线程开始");
			/*生成随机订单号,放入消息队列中*/
			String orderNumber = RandomStringUtils.randomNumeric(8);
			mockQueue.setPlaceOrder(orderNumber);
			/*处理订单*/
			DeferredResult<String> result = new DeferredResult<>();
			deferredResultHolder.getMap().put(orderNumber, result);
			logger.info("主线程返回");
			return result;
		}
	}

**模拟下单监听过程-线程：**

	package com.zhqx.web.async;
	
	import org.apache.commons.lang.StringUtils;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.ApplicationListener;
	import org.springframework.context.event.ContextRefreshedEvent;
	import org.springframework.stereotype.Component;
	
	//消息队列监听器
	//用来监听消息队列中,是否有订单处理完成的消息
	//ContextRefreshedEvent:整个Spring容器初始化完毕的事件
	@Component
	public class QueueListener implements ApplicationListener<ContextRefreshedEvent> {
	
		private Logger logger = LoggerFactory.getLogger(getClass());
	
		@Autowired
		private MockQueue mockQueue;
		@Autowired
		private DeferredResultHolder deferredResultHolder;
	
		@Override
		public void onApplicationEvent(ContextRefreshedEvent event) {
			// 将循环放入线程中,防止主线程阻死
			new Thread(() -> {
				while (true) {
					// 当队列中表示订单完成的值不为空时,表示有订单处理完成,处理订单结果
					if (StringUtils.isNotBlank(mockQueue.getCompleteOrder())) {
						String orderNumber = mockQueue.getCompleteOrder();
						logger.info("返回订单处理结果:" + orderNumber);
						deferredResultHolder.getMap().get(orderNumber).setResult("place order success");
						// 订单处理完成后,更改订单处理状态
						mockQueue.setCompleteOrder(null);
					} else {
						// 如果没有监听到处理完毕的值,让线程沉睡100毫秒,再次执行监听功能
						try {
							Thread.sleep(100);
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				}
			}).start();
		}
	}

启动服务，在浏览器中访问：`http://localhost:8080/order`,返回结果包含如下内容：

	17:15:07.208  INFO 19696 --- [nio-8080-exec-1] com.zhqx.web.async.AsyncController       : 主线程开始
	17:15:07.210  INFO 19696 --- [nio-8080-exec-1] com.zhqx.web.async.AsyncController       : 主线程返回
	17:15:07.210  INFO 19696 --- [      Thread-10] com.zhqx.web.async.MockQueue             : 接到下单请求, 69276697
	17:15:08.210  INFO 19696 --- [      Thread-10] com.zhqx.web.async.MockQueue             : 下单请求处理完毕,69276697
	17:15:08.264  INFO 19696 --- [       Thread-7] com.zhqx.web.async.QueueListener         : 返回订单处理结果:69276697

可以看到程序有3个线程参与订单的处理。

**异步处理，线程配置。**

默认情况下，`Springboot`使用`SimpleAsyncTaskExecutor`来管理线程，它是不会重用线程的，每次都会重开一个线程来处理。	

所以我们需要自定义异步任务线程池。

### 九、使用Swagger自动生成html文档 ###

在`zhqx-security-demo`项目的pom文件中引入相关jar包：

	<dependency>
		<groupId>io.springfox</groupId>
		<artifactId>springfox-swagger2</artifactId>
		<version>2.9.2</version>
	</dependency>
	<dependency>
		<groupId>io.springfox</groupId>
		<artifactId>springfox-swagger-ui</artifactId>
		<version>2.9.2</version>
	</dependency>

在启动类`DemoApplication`上加上`@EnableSwagger2`注解，表示启动Swagger。启动服务。

在浏览器地址栏中输入：`http://localhost:8080/swagger-ui.html`，即可看到生效了。

`@ApiOperation`注解，用来描述方法的作用。

`@ApiModelProperty`注解，用来注解方法参数，如果方法参数封装在一个对象上，那么在这个对象上使用该注解。

`@ApiParam`注解，用来注解方法参数，如果方法参数不是封装在对象中，那么使用该注解来注释参数含义。

下面是相关注解的代码片段：

**@ApiOperation注解：**

	@GetMapping
	@JsonView(User.UserSimpleView.class)
	@ApiOperation("用户查询服务")
	public List<User> query(UserQueryCondition condition,@PageableDefault(page = 2, size = 17, sort = "username,asc") Pageable pageable) {
		
		System.out.println(ReflectionToStringBuilder.toString(condition, ToStringStyle.MULTI_LINE_STYLE));
		
		System.out.println(pageable.getPageSize());
		System.out.println(pageable.getPageNumber());
		System.out.println(pageable.getSort());
		
		List<User> users = new ArrayList<User>();
		users.add(new User());
		users.add(new User());
		users.add(new User());
		return users;
	}


**@ApiModelProperty注解：**

	public class UserQueryCondition {
		
		private String username;
		
		@ApiModelProperty(value = "用户年龄起始值")
		private int age;
		
		@ApiModelProperty(value = "用户年龄终止值")
		private int ageTo;
		
		private String xxx;

		......

**@ApiParam注解：**

	@GetMapping("/{id:\\d+}")
	@JsonView(User.UserDetailView.class)
	public User getInfo(@ApiParam("用户id") @PathVariable String id) {
		System.out.println("进入getInfo方法");
		//throw new RuntimeException("user not exist");
		//throw new UserNotExistException(id);
		User user = new User();
		user.setUsername("tom");
		return user;
	}

### 十、使用WireMock伪造REST服务 ###

这个实际上也是对外提供一个API服务。大部分情况下用Swagger就可以了。这种方法前期准备比较多。