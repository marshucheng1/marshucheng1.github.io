---
layout: post
title: springcloud（二）
date: 2021-04-17 09:11:22
categories: springcloud
tags: springcloud
author: MarsHu
---

* content
{:toc}

### 集成持久层框架Mybatis  ###
使用Navicat创建本地Mysql数据库course.

	CREATE DATABASE `course` CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_general_ci';

使用Navicat创建用户

![springcloud2-1.png](http://marshucheng1.github.io/assets/springcloud/springcloud2-1.png)







对应的SQL语句如下:

	CREATE USER `course`@`localhost` IDENTIFIED WITH mysql_native_password BY 'course';

创建用户时,如果需要该用户即可以本地连接,又可以外网连接,则在主机那一栏应该填写成%。

使用Navicat分配权限:

![springcloud2-2.png](http://marshucheng1.github.io/assets/springcloud/springcloud2-2.png)

对应的SQL语句如下:

	GRANT Alter, Alter Routine, Create, Create Routine, Create Temporary Tables, Create View, Delete, Drop, Event, Execute, Grant Option, Index, Insert, Lock Tables, References, Select, Show View, Trigger, Update ON `course`.* TO `course`@`localhost`;

使用刚刚创建的用户连接mysql,该用户只能看到2个库,并且初始访问时报错.

![springcloud2-3.png](http://marshucheng1.github.io/assets/springcloud/springcloud2-3.png)

初始访问报错,是因为course库中一张表都没有,所以我们可以创建一张test表:

	CREATE TABLE `test` (
	  `id` varchar(255) NOT NULL,
	  `name` varchar(255) DEFAULT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

创建好test表后,关闭之前的连接,再次使用course用户连接mysql就不会出错了.为test表新增一条数据:

	INSERT INTO `course`.`test` (`id`, `name`) VALUES ('1', 'test');

修改父module项目course下的pom.xml,引入mybatis和mysql依赖:

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	    <packaging>pom</packaging>
	    <modules>
	        <module>eureka</module>
	        <module>system</module>
	        <module>gateway</module>
	    </modules>
	    <parent>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-parent</artifactId>
	        <version>2.5.3</version>
	        <relativePath/> <!-- lookup parent from repository -->
	    </parent>
	    <groupId>com.zhqx</groupId>
	    <artifactId>course</artifactId>
	    <version>0.0.1-SNAPSHOT</version>
	    <name>course</name>
	    <description>Demo project for Spring Boot</description>
	    <properties>
	        <java.version>1.8</java.version>
	        <spring-cloud.version>2020.0.3</spring-cloud.version>
	    </properties>
	
	    <dependencyManagement>
	        <dependencies>
	            <dependency>
	                <groupId>org.springframework.cloud</groupId>
	                <artifactId>spring-cloud-dependencies</artifactId>
	                <version>${spring-cloud.version}</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	
	            <!-- 集成mybatis -->
	            <dependency>
	                <groupId>org.mybatis.spring.boot</groupId>
	                <artifactId>mybatis-spring-boot-starter</artifactId>
	                <version>2.2.0</version>
	            </dependency>
	            <dependency>
	                <groupId>mysql</groupId>
	                <artifactId>mysql-connector-java</artifactId>
	                <version>5.1.47</version>
	            </dependency>
	        </dependencies>
	    </dependencyManagement>
	
	    <build>
	        <plugins>
	            <plugin>
	                <groupId>org.springframework.boot</groupId>
	                <artifactId>spring-boot-maven-plugin</artifactId>
	            </plugin>
	        </plugins>
	    </build>
	
	</project>

在子module项目system中引入mybaits和mysql依赖,因为父module中已经指定版本,所以子module中不需要指定版本号了:

	<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

引入mysql的jar包后,需要在配置文件中配置数据库连接,否则会报错,修改system项目的application.properties,新增如下配置:

	spring.datasource.driver-class-name=com.mysql.jdbc.Driver
	spring.datasource.url=jdbc:mysql://localhost:3306/course?characterEncoding=utf-8&serverTimezone=GMT%2B8&autoReconnect=true&useSSL=false
	spring.datasource.username=course
	spring.datasource.password=course

新增Test类,对应之前建的数据库表test,

	package com.course.system.domain;
	
	public class Test {
	
	    private String id;
	    private String name;
	
	    //省略其他方法...
	}

新增TestMapper,

	package com.course.system.mapper;
	
	import com.course.system.domain.Test;
	
	import java.util.List;
	
	public interface TestMapper {
	    List<Test> list();
	}

新增TestService,

	package com.course.system.service;
	
	import com.course.system.domain.Test;
	import com.course.system.mapper.TestMapper;
	import org.springframework.stereotype.Service;
	
	import javax.annotation.Resource;
	import java.util.List;
	
	@Service
	public class TestService {
	
	    @Resource
	    private TestMapper mapper;
	
	    public List<Test> list() {
	        return mapper.list();
	    }
	}

在resources目录下新建文件夹mapper,并在其中新增TestMapper.xml

	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE mapper
	        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
	<mapper namespace="com.course.system.mapper.TestMapper">
	
	    <select id="list" resultType="com.course.system.domain.Test">
	        select `id`, `name` from `test`
	    </select>
	
	</mapper>

修改TestController类:

	package com.course.system.controller;
	
	import com.course.system.domain.Test;
	import com.course.system.service.TestService;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	import javax.annotation.Resource;
	import java.util.List;
	
	@RestController
	public class TestController {
	
	    @Resource
	    private TestService testService;
	
	    @RequestMapping("/test")
	    public List<Test> test() {
	        return testService.list();
	    }
	}

**在启动类SystemApplication上加上注解`@MapperScan("com.course.system.mapper")`,扫描mybaits接口,在配置文件application.properties中新增配置`mybatis.mapper-locations=classpath:/mapper/*.xml`指定mapper接口对应的xml文件,**

上面操作完成后,依次启动eureka、system、gateway,浏览器访问:http://127.0.0.1:9000/system/test,得到如下结果:

	[
	{
	id: "1",
	name: "test"
	}
	]

### 项目热部署  ###
在system项目中增加依赖

	<!-- 热部署DevTools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>

进入File->Settings->Build,Execution,Deployment->Compiler,勾选Build project automatically

![springcloud2-4.png](http://marshucheng1.github.io/assets/springcloud/springcloud2-4.png)

在idea中连续按下shift,在弹出窗口中输入registry,然后点击actions下的Registry...

![springcloud2-5.png](http://marshucheng1.github.io/assets/springcloud/springcloud2-5.png)

在弹出窗口中,勾选compiler.automake.allow.when.app.running:

![springcloud2-6.png](http://marshucheng1.github.io/assets/springcloud/springcloud2-6.png)

在增加热部署依赖时,可以重新导入下maven依赖,否则可能会出现jar包没有导入的情况,从而导致失效,当我们修改完代码后,可以手动ctrl+s保存下代码,加快热部署进度。

在system项目的配置文件application.properties中增加mybatisSQL打印的配置:

	logging.level.com.course.system.mapper=trace


### 搭建服务模块-server  ###
当前我们将Test类相关的业务都放在system项目中,假设后面我们又新增了新的子module项目,其中也需要用到Test相关业务,那我们就需要进行大量的业务复制,所以我们可以新增一个业务服务模块,将基本的业务放到其中,这样后续有其他子module需要的话,只需要引用该模块即可。

按照之前的操作,新增module-server项目,并将system项目中Test相关业务移入其中,并将代码修改正确,整个项目结构如下:

![springcloud2-7.png](http://marshucheng1.github.io/assets/springcloud/springcloud2-7.png)

server项目去除了eureka的相关依赖,pom.xml文件如下:

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <parent>
	        <artifactId>course</artifactId>
	        <groupId>com.zhqx</groupId>
	        <version>0.0.1-SNAPSHOT</version>
	    </parent>
	    <modelVersion>4.0.0</modelVersion>
	
	    <artifactId>server</artifactId>
	
	    <dependencies>
	
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-web</artifactId>
	        </dependency>
	
	        <!-- 集成mybatis -->
	        <dependency>
	            <groupId>org.mybatis.spring.boot</groupId>
	            <artifactId>mybatis-spring-boot-starter</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>mysql</groupId>
	            <artifactId>mysql-connector-java</artifactId>
	        </dependency>
	
	        <!-- 热部署DevTools -->
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-devtools</artifactId>
	        </dependency>
	    </dependencies>
	
	</project>

在course父项目的pom.xml中保留原有依赖关系,增加server项目的依赖,

	<dependency>
                <groupId>com.zhqx</groupId>
                <artifactId>server</artifactId>
                <version>0.0.1-SNAPSHOT</version>
            </dependency>


在子项目system的pom.xml中中保留原有依赖关系,增加sever依赖,不需要指定版本号了:

	<dependency>
                <groupId>com.zhqx</groupId>
                <artifactId>server</artifactId>
            </dependency>

**在system中引入server依赖时,不指定版本号的话,在有的maven版本里面可能是没办法引入的,换一个maven版本吧(案例使用maven3.5.0)**

修改system项目中的TestController:

	package com.course.system.controller;
	
	import com.course.server.domain.Test;
	import com.course.server.service.TestService;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	import javax.annotation.Resource;
	import java.util.List;
	
	@RestController
	public class TestController {
	
	    @Resource
	    private TestService testService;
	
	    @RequestMapping("/test")
	    public List<Test> test() {
	        return testService.list();
	    }
	}

修改SystemApplication:

	package com.course.system;
	
	import org.mybatis.spring.annotation.MapperScan;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
	import org.springframework.core.env.ConfigurableEnvironment;
	
	@SpringBootApplication
	@EnableEurekaClient
	@ComponentScan("com.course")
	@MapperScan("com.course.server.mapper")
	public class SystemApplication {
	
	    private static final Logger LOG = LoggerFactory.getLogger(SystemApplication.class);
	
	    public static void main(String[] args) {
	        SpringApplication app = new SpringApplication(SystemApplication.class);
	        ConfigurableEnvironment env = app.run(args).getEnvironment();
	        LOG.info("启动成功！！");
	        LOG.info("System地址: \thttp://127.0.0.1:{}", env.getProperty("server.port"));
	    }
	
	}

在这里如果不加上`@ComponentScan("com.course")`注解,会出现错误:

	A component required a bean of type 'com.course.server.service.TestService' that could not be found.

经过修改后的system项目整体结构:

![springcloud2-8.png](http://marshucheng1.github.io/assets/springcloud/springcloud2-8.png)

将公共配置信息提取到sever项目中,在server项目的src/main/resources目录下新建config文件夹,将application.properties放入其中:

	#增加数据库连接
	spring.datasource.driver-class-name=com.mysql.jdbc.Driver
	spring.datasource.url=jdbc:mysql://localhost:3306/course?characterEncoding=utf-8&serverTimezone=GMT%2B8&autoReconnect=true&useSSL=false
	spring.datasource.username=course
	spring.datasource.password=course
	
	mybatis.mapper-locations=classpath:/mapper/*.xml
	
	logging.level.com.course.server.mapper=trace

修改system项目的配置文件application.properties,移除公共配置部分,移除后配置如下:

	spring.application.name=system
	server.servlet.context-path=/system
	server.port=9001
	eureka.client.service-url.defaultZone=http://localhost:8761/eureka/

### 集成mybatis generator  ###
在父项目course的pom.xml文件中新增mybatis-generator插件:

	<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <!--  mybatis generator 自动生成代码插件  -->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.7</version>
                <configuration>
                    <configurationFile>src/main/resources/generator/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>5.1.47</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>

在子项目sever的src/main/resources目录下新增文件夹generator,并在该文件夹下新增配置文件generatorConfig.xml:

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE generatorConfiguration
	        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
	        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
	
	<generatorConfiguration>
	    <context id="Mysql" targetRuntime="MyBatis3" defaultModelType="flat">
	
	        <property name="autoDelimitKeywords" value="true"/>
	        <property name="beginningDelimiter" value="`"/>
	        <property name="endingDelimiter" value="`"/>
	
	        <!--覆盖生成XML文件，不需要每次手动删除TestMapper.xml了-->
	        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />
	        <!-- 生成的实体类添加toString()方法 -->
	        <plugin type="org.mybatis.generator.plugins.ToStringPlugin" />
	
	        <!-- 不生成注释 -->
	        <commentGenerator>
	            <property name="suppressAllComments" value="true"/>
	        </commentGenerator>
	
	        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
	                        connectionURL="jdbc:mysql://localhost:3306/course"
	                        userId="course"
	                        password="course">
	        </jdbcConnection>
	
	        <!-- domain类的位置 -->
	        <javaModelGenerator targetProject="src/main/java"
	                            targetPackage="com.course.server.domain"/>
	
	        <!-- mapper xml的位置 -->
	        <sqlMapGenerator targetProject="src/main/resources"
	                         targetPackage="mapper"/>
	
	        <!-- mapper类的位置 -->
	        <javaClientGenerator targetProject="src/main/java"
	                             targetPackage="com.course.server.mapper"
	                             type="XMLMAPPER" />
	
	
	        <table tableName="test" domainObjectName="Test"/>
	    </context>
	</generatorConfiguration>

按照下图顺序操作,配置mybaitis-maven生成命令:

![springcloud2-9.png](http://marshucheng1.github.io/assets/springcloud/springcloud2-9.png)

![springcloud2-10.png](http://marshucheng1.github.io/assets/springcloud/springcloud2-10.png)


配置正常后,我们就可以通过自动生成工具帮我们生成相关代码了。修改server项目中的TestService类,让代码正常:

	package com.course.server.service;
	
	import com.course.server.domain.Test;
	import com.course.server.domain.TestExample;
	import com.course.server.mapper.TestMapper;
	import org.springframework.stereotype.Service;
	
	import javax.annotation.Resource;
	import java.util.List;
	
	@Service
	public class TestService {
	
	    @Resource
	    private TestMapper mapper;
	
	    public List<Test> list() {
			//select id, `name` from test WHERE ( id = ? ) order by id desc
	        TestExample testExample = new TestExample();
	        testExample.createCriteria().andIdEqualTo("1");//where条件
	        testExample.setOrderByClause("id desc");//排序
	        return mapper.selectByExample(testExample);
	    }
	}