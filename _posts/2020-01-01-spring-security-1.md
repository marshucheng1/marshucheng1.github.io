---
layout: post
title: Spring Security开发安全的REST服务（一）
date: 2020-01-01 13:31:22
categories: JavaWeb
tags: springboot
author: MarsHu
---

* content
{:toc}

### 项目环境搭建 ###
这一章，我们会搭建和配置开发环境，并写一个helloworld程序，用以测试是否搭建成功，对整体架构有所了解。

开发环境安装：1.JDK安装，2.STS(或者其他开发工具都可以)，3.安装MySql。

这里就不带大家了解如何安装了，基本功，直接略过。





### 代码结构介绍 ###
项目为多模块maven项目，由5个模块组成。以下是各个模块介绍：

zhqx-security：主模块，包含下面4各模块

zhqx-security-core：核心业务逻辑

zhqx-security-browser：浏览器安全特定代码

zhqx-security-app：app相关特定代码

zhqx-security-demo：样例程序


### zhqx-security模块pom文件 ###
使用platform-bom（Spring io平台）以及spring-cloud作为父级依赖。注意spring官方的文档说明。

最新的io平台已经更新到`springboot2.0.8`版本。选择相对应的springcloud版本。

	<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.zhqx.security</groupId>
	<artifactId>zhqx-security</artifactId>
	<version>1.0.0-SNAPSHOT</version>
	<packaging>pom</packaging>

	<properties>
		<zhqx.security.version>1.0.0-SNAPSHOT</zhqx.security.version>
	</properties>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>io.spring.platform</groupId>
				<artifactId>platform-bom</artifactId>
				<version>Cairo-SR7</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Finchley.SR4</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
		</plugins>
	</build>

	<modules>
		<module>../zhqx-security-app</module>
		<module>../zhqx-security-browser</module>
		<module>../zhqx-security-core</module>
		<module>../zhqx-security-demo</module>
	</modules>
	</project>

### zhqx-security-core模块pom文件 ###
这里需要注意的是，在高版本的springcloud版本中，直接使用`spring-cloud-starter-oauth2`是无法启动后面的springboot启动类的。

如果要确保正常启动，还需要引入`spring-boot-starter-data-web`。

	<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<artifactId>zhqx-security-core</artifactId>
	<parent>
		<groupId>com.zhqx.security</groupId>
		<artifactId>zhqx-security</artifactId>
		<version>1.0.0-SNAPSHOT</version>
		<relativePath>../zhqx-security</relativePath>
	</parent>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-oauth2</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.social</groupId>
			<artifactId>spring-social-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.social</groupId>
			<artifactId>spring-social-core</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.social</groupId>
			<artifactId>spring-social-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.social</groupId>
			<artifactId>spring-social-web</artifactId>
		</dependency>
		<dependency>
			<groupId>commons-lang</groupId>
			<artifactId>commons-lang</artifactId>
		</dependency>
		<dependency>
			<groupId>commons-collections</groupId>
			<artifactId>commons-collections</artifactId>
		</dependency>
		<dependency>
			<groupId>commons-beanutils</groupId>
			<artifactId>commons-beanutils</artifactId>
		</dependency>
	</dependencies>
	</project>

### zhqx-security-browser模块pom文件 ###

	<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<artifactId>zhqx-security-browser</artifactId>
	<parent>
		<groupId>com.zhqx.security</groupId>
		<artifactId>zhqx-security</artifactId>
		<version>1.0.0-SNAPSHOT</version>
		<relativePath>../zhqx-security</relativePath>
	</parent>

	<dependencies>
		<dependency>
			<groupId>com.zhqx.security</groupId>
			<artifactId>zhqx-security-core</artifactId>
			<version>${zhqx.security.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session</artifactId>
			<version>1.3.4.RELEASE</version>
		</dependency>
	</dependencies>
	</project>

### zhqx-security-app模块pom文件 ###

	<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<artifactId>zhqx-security-app</artifactId>
	<parent>
		<groupId>com.zhqx.security</groupId>
		<artifactId>zhqx-security</artifactId>
		<version>1.0.0-SNAPSHOT</version>
		<relativePath>../zhqx-security</relativePath>
	</parent>

	<dependencies>
		<dependency>
			<groupId>com.zhqx.security</groupId>
			<artifactId>zhqx-security-core</artifactId>
			<version>${zhqx.security.version}</version>
		</dependency>
	</dependencies>
	</project>

### zhqx-security-demo模块pom文件 ###

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
	</dependencies>
	</project>

demo项目的启动类代码：

	package com.zhqx;
	
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	@SpringBootApplication
	@RestController
	@EnableAutoConfiguration(exclude = {
			org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration.class
	})
	public class DemoApplication {
		public static void main(String[] args) {
			SpringApplication.run(DemoApplication.class, args);
		}
		
		@GetMapping("/hello")
		public String hello() {
			return "hello spring security";
		}
	}

这里个注解的意思是，默认启动时，选择关闭spring-security安全验证。【springboot2.x不能通过配置文件中关闭security安全验证】。

	@EnableAutoConfiguration(exclude = {
				org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration.class
		})

当然我们也可以不使用这个注解，我们可以单独构建自己的security安全配置。代码如下：

	package com.zhqx.config;
	
	import org.springframework.context.annotation.Configuration;
	import org.springframework.security.config.annotation.web.builders.HttpSecurity;
	import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
	import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
	
	@Configuration
	@EnableWebSecurity
	public class SecurityConfig extends WebSecurityConfigurerAdapter {
	    /**
	     * spring boot1.x配置security关闭http基本验证,只需要在application.yml中配置
	     * security.basic.enabled=false即可，
	     * 但是spring boot 2.0+之后这样配置就不能生效了。
	     * 但是我们可以在代码中去配置。
	     * 我们可以新建一个类SecurityConfig 继承WebSecurityConfigurerAdapter类，
	     * 然后重写父类中的configure(HttpSecurity http) 方法。
	     *
	     * @param http
	     * @throws Exception
	     */
	    @Override
	    protected void configure(HttpSecurity http) throws Exception {
			http.csrf().disable().authorizeRequests().anyRequest().permitAll().and().logout().permitAll();
	    }
	}

demo项目的yml配置文件：

	spring:
	  #session管理
	  session:
	    store-type: none
	  #mysql数据库连接配置
	  datasource:
	    driver-class-name: com.mysql.jdbc.Driver
	    url: jdbc:mysql://localhost:3306/security_demo?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=UTC
	    username: root
	    password: root