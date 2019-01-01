---
layout: post
title: JAVA-jdbc+mysql+servlet分页
date: 2017-02-14 15:33:43
categories: WEB
tags: servlet
author: MarsHu
---

* content
{:toc}

# 数据库配置文件properties #
使用`properties`文件存放数据库配置，读取`properties`文件。并配置`dbcp`连接池。

创建DBUtil连接，连接数据库，方便复用建立数据库的连接和关闭操作。

> **数据库相关配置，db.properties**

	jdbc.driver=com.mysql.jdbc.Driver
	jdbc.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false
	jdbc.user=root
	jdbc.password=123456





# 工具类DBUtil #
> **配置了连接池的DBUtil.java完整代码**

	package cn.ahzhuoshan.util;

	import java.io.IOException;
	import java.sql.Connection;
	import java.sql.SQLException;
	import java.util.Properties;
	import org.apache.commons.dbcp.BasicDataSource;

	public class DBUtil {
		private static BasicDataSource dataSource = null;

		public static void init() {
			Properties dbProps = new Properties();
			// 取配置文件可以根据实际的不同修改
			try {
				dbProps.load(DBUtil.class.getClassLoader().getResourceAsStream("db.properties"));
			} catch (IOException e) {
				e.printStackTrace();
			}
			try {
				String driveClassName = dbProps.getProperty("jdbc.driver");
				String url = dbProps.getProperty("jdbc.url");
				String username = dbProps.getProperty("jdbc.user");
				String password = dbProps.getProperty("jdbc.password");
				String initialSize = dbProps.getProperty("dataSource.initialSize");
				String minIdle = dbProps.getProperty("dataSource.minIdle");
				String maxIdle = dbProps.getProperty("dataSource.maxIdle");
				String maxWait = dbProps.getProperty("dataSource.maxWait");
				String maxActive = dbProps.getProperty("dataSource.maxActive");
				dataSource = new BasicDataSource();
				dataSource.setDriverClassName(driveClassName);
				dataSource.setUrl(url);
				dataSource.setUsername(username);
				dataSource.setPassword(password);
				// 初始化连接数
				if (initialSize != null) {
					dataSource.setInitialSize(Integer.parseInt(initialSize));
				}
				// 最小空闲连接
				if (minIdle != null) {
					dataSource.setMinIdle(Integer.parseInt(minIdle));
				}
				// 最大空闲连接
				if (maxIdle != null) {
					dataSource.setMaxIdle(Integer.parseInt(maxIdle));
				}
				// 超时回收时间(以毫秒为单位)
				if (maxWait != null) {
					dataSource.setMaxWait(Long.parseLong(maxWait));
				}
				// 最大连接数
				if (maxActive != null) {
					if (!maxActive.trim().equals("0")) {
						dataSource.setMaxActive(Integer.parseInt(maxActive));
					}
				}
			} catch (Exception e) {
				e.printStackTrace();
				System.out.println("创建连接池失败!请检查设置!");
			}
		}

		public static synchronized Connection getConnection() throws SQLException {
			Connection conn = null;
			if (dataSource == null) {
				init();
			}
			if (dataSource != null) {
				conn = dataSource.getConnection();
			}
			return conn;
		}

		public static void main(String[] args) throws SQLException {
			Connection conn = DBUtil.getConnection();
			System.out.println(conn);
		}
	}

# 实体类User #

	package cn.ahzhuoshan.entity;

	import java.io.Serializable;

	public class User implements Serializable {
		private int id;
	
		private String name;
		
		private int age;
	
		private String email;
	
		public User() {
		}

		public User(String name, int age, String email) {
			this.name = name;
			this.age = age;
			this.email = email;
		}

		public int getId() {
			return id;
		}

		public void setId(int id) {
			this.id = id;
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

		public String getEmail() {
			return email;
		}

		public void setEmail(String email) {
			this.email = email;
		}

		public String toString() {
			return "User [id=" + id + ", name=" + name + ", age=" + age + ", email=" + email + "]";
		}
	
	}

# 分页工具类Page #
	
	package cn.ahzhuoshan.util;

	import java.util.List;
	import cn.ahzhuoshan.entity.User;

	public class Page {
		// 总页数
		private int totalPageCount = 0;
		// 页面大小，即每页显示记录数
		private int pageSize = 5;
		// 记录总数
		private int totalCount;
		// 当前页码
		private int currPageNo = 1;
		// 每页用户集合
		private List<User> usersList;

		public int getCurrPageNo() {
			if (totalPageCount == 0) {
				return 0;
			}
			return currPageNo;
		}

		public void setCurrPageNo(int currPageNo) {
			if (currPageNo > 0) {
				this.currPageNo = currPageNo;
			}
		}

		public int getPageSize() {
			return pageSize;
		}

		public void setPageSize(int pageSize) {
			if (pageSize > 0) {
				this.pageSize = pageSize;
			}
		}

		public int getTotalCount() {
			return totalCount;
		}

		public void setTotalCount(int totalCount) {
			if (totalCount > 0) {
				this.totalCount = totalCount;
				// 计算总页数
				totalPageCount = this.totalCount % pageSize == 0 ? (this.totalCount / pageSize)
					: (this.totalCount / pageSize + 1);
			}
		}

		public int getTotalPageCount() {
			return totalPageCount;
		}

		public void setTotalPageCount(int totalPageCount) {
			this.totalPageCount = totalPageCount;
		}

		public List<User> getUsersList() {
			return usersList;
		}

		public void setUsersList(List<User> usersList) {
			this.usersList = usersList;
		}
	
	}

# JDBC连接数据库读取数据 #
> **UserDao接口**

	package cn.ahzhuoshan.dao;

	import java.util.List;
	import cn.ahzhuoshan.entity.User;

	public interface UserDao {
		// 获得所有用户
		List<User> getAll() throws Exception;

		// 获得用户总数
		public int getTotalCount() throws Exception;

		// 分页获得用户
		List<User> getPageList(int currPageNo, int pageSize) throws Exception;
	}

> **UserDao接口jdbc实现**

	package cn.ahzhuoshan.dao;

	import java.sql.Connection;
	import java.sql.PreparedStatement;
	import java.sql.ResultSet;
	import java.util.ArrayList;
	import java.util.List;
	import cn.ahzhuoshan.entity.User;
	import cn.ahzhuoshan.util.DBUtil;

	public class UserDaoImpl implements UserDao {

		private Connection conn;
		private PreparedStatement stat;
		private ResultSet rs;

		public UserDaoImpl() {
			try {
				conn = DBUtil.getConnection();
			} catch (Exception e) {
				e.printStackTrace();
			}
		}

		public List<User> getAll() throws Exception {
			List<User> users = new ArrayList<User>();
			User user = null;
			String sql = "select id, name, age, email from user";
			try {
				stat = conn.prepareStatement(sql);
				rs = stat.executeQuery();
				while (rs.next()) {
					user = new User();
					user.setId(rs.getInt("id"));
					user.setName(rs.getString("name"));
					user.setAge(rs.getInt("age"));
					user.setEmail(rs.getString("email"));
					users.add(user);
				}
			} catch (Exception e) {
				e.printStackTrace();
				throw e;
			}
			return users;
		}

		public int getTotalCount() throws Exception {
			// 比select * from user效率高...
			String sql = "select count(id) from user";
			int count = -1;
			try {
				stat = conn.prepareStatement(sql);
				rs = stat.executeQuery();
				rs.next();
				count = rs.getInt(1);
			} catch (Exception e) {
				e.printStackTrace();
				throw e;
			}
			return count;
		}

		public List<User> getPageList(int currPageNo, int pageSize) throws Exception {
			List<User> users = new ArrayList<User>();
			// limit 0, 5,查询的结果是1-5;如果是limit 1, 5,查询的结果是2-6
			String sql = "select id, name, age, email from user limit ?, ?";
			try {
				stat = conn.prepareStatement(sql);
				stat.setInt(1, (currPageNo - 1) * pageSize);
				stat.setInt(2, pageSize);
				rs = stat.executeQuery();
				User user = null;
				while (rs.next()) {
					user = new User();
					user.setId(rs.getInt("id"));
					user.setName(rs.getString("name"));
					user.setAge(rs.getInt("age"));
					user.setEmail(rs.getString("email"));
					users.add(user);
				}
			} catch (Exception e) {
				e.printStackTrace();
				throw e;
			}
			return users;
		}

	}

# 用户管理业务层service #
> **UserService接口**

	package cn.ahzhuoshan.service;

	import java.util.List;
	import cn.ahzhuoshan.entity.User;
	import cn.ahzhuoshan.util.Page;

	public interface UserService {
		// 获取所有用户
		List<User> findAllUsers() throws Exception;

		// 分页获取用户
		List<User> findPageUsers(Page page) throws Exception;
	}

> **UserService接口实现**

	package cn.ahzhuoshan.service;

	import java.util.List;
	import cn.ahzhuoshan.dao.UserDao;
	import cn.ahzhuoshan.dao.UserDaoImpl;
	import cn.ahzhuoshan.entity.User;
	import cn.ahzhuoshan.util.Page;

	public class UserServiceImpl implements UserService {

		// 获取所有用户
		public List<User> findAllUsers() throws Exception {
			UserDao dao = new UserDaoImpl();
			List<User> users = dao.getAll();
			return users;
		}

		// 分页获取用户
		public List<User> findPageUsers(Page page) throws Exception {
			UserDao dao = new UserDaoImpl();
			// 查询用户总数设置给分页工具类Page
			page.setTotalCount(dao.getTotalCount());
			List<User> users = dao.getPageList(page.getCurrPageNo(), page.getPageSize());
			return users;
		}

	}

# 用户管理控制器Servlet #

	package cn.ahzhuoshan.action;

	import java.io.IOException;
	import java.io.PrintWriter;
	import java.util.List;
	import javax.servlet.ServletException;
	import javax.servlet.http.HttpServlet;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import cn.ahzhuoshan.entity.User;
	import cn.ahzhuoshan.service.UserService;
	import cn.ahzhuoshan.service.UserServiceImpl;
	import cn.ahzhuoshan.util.Page;

	public class UserServlet extends HttpServlet {
		private static final long serialVersionUID = 1L;

		public void service(HttpServletRequest request, 
			HttpServletResponse response) throws ServletException, IOException {
			// 处理编码问题
			request.setCharacterEncoding("UTF-8");
			response.setContentType("text/html;charset=UTF-8");
			// 获取输出流
			PrintWriter out = response.getWriter();

			// 获取请求资源路径
			String uri = request.getRequestURI();
			// 获取请求资源路径中除应用名以外的部分
			String action = uri.substring(uri.lastIndexOf("/") + 1, uri.lastIndexOf("."));

			if (action.equals("list")) {
				try {
					UserService service = new UserServiceImpl();
					// 初始化分页工具类
					Page page = new Page();
					// 获取页面currPageNo
					String currPageNo = request.getParameter("currPageNo");
					if (currPageNo == null || currPageNo == "") {
						// 第一次查询没有currPageNo参数,默认值1
						List<User> users = service.findPageUsers(page);
						page.setUsersList(users);
						request.setAttribute("page", page);
						request.getRequestDispatcher("index.jsp").forward(request, response);
					} else {
						// 分页查询
						page.setCurrPageNo(Integer.parseInt(currPageNo));
						List<User> users = service.findPageUsers(page);
						page.setUsersList(users);
						request.setAttribute("page", page);
						request.getRequestDispatcher("index.jsp").forward(request, response);
					}
				} catch (Exception e) {
					e.printStackTrace();
					out.print("系统繁忙");
				}
			}
		}

	}

# 用户管理分页显示界面index.jsp #

	<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
	<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
	<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
	<html>
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
	<title>用户列表</title>
	<style type="text/css">
	body, table {
		margin: 0px auto;
		text-align: center;
	}
	</style>
	</head>
	<body>
		<table border="1px" cellpadding="0" cellspacing="0" width="300px" height="200px">
			<caption>用户管理</caption>
			<tr>
				<td>id</td>
				<td>姓名</td>
				<td>年龄</td>
				<td>邮箱</td>
			</tr>
			<c:forEach var="user" items="${page.usersList}" varStatus="s">
				<tr>
					<td>${user.id }</td>
					<td>${user.name }</td>
					<td>${user.age }</td>
					<td>${user.email }</td>
				</tr>
			</c:forEach>
		</table>
		<!-- 当currPageNo<=1时,没有上一页链接 -->
		<c:if test="${page.currPageNo gt 1 }">
			<a href="list.do?currPageNo=${page.currPageNo-1 }">上一页</a>
		</c:if>
		&nbsp;&nbsp;当前页:${page.currPageNo }&nbsp;/&nbsp;总页数:${page.totalPageCount }&nbsp;&nbsp;
		<!-- 当currPageNo>总页数时,没有下一页链接 -->
		<c:if test="${page.currPageNo lt page.totalPageCount }">
			<a href="list.do?currPageNo=${page.currPageNo+1 }">下一页</a>
		</c:if>
	</body>
	</html>

# 演示demo需要导入的jar包 #

	mysql-connector-java-5.1.46.jar
	commons-dbcp-1.4.jar
	commons-pool-1.5.4.jar
	jstl-1.2.jar

> **maven配置**

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.46</version>
    </dependency>
    <dependency>
        <groupId>commons-dbcp</groupId>
        <artifactId>commons-dbcp</artifactId>
        <version>1.4</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>

**需要添加对应版本的`Targeted Runtimes`也即是tomcat运行环境，如图**
![servletpage.png](http://marshucheng1.github.io/assets/servletpage.png)