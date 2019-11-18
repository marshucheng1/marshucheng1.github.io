---
layout: post
title: MyBatis将数据库中的任意两个字段映射为map集合的key和value
date: 2017-03-14 22:33:43
categories: java
tags: mybatis
author: MarsHu
---

* content
{:toc}

# 数据库表数据 #
	CREATE TABLE `food_new` (
	  `id` int(11) NOT NULL AUTO_INCREMENT,
	  `code` varchar(255) DEFAULT NULL COMMENT '物料编码',
	  `name` varchar(255) NOT NULL COMMENT '物料名称',
	  `type` varchar(255) DEFAULT NULL COMMENT '规格型号',
	  `unit` varchar(255) DEFAULT NULL COMMENT '单位',
	  `price` double(10,2) DEFAULT NULL COMMENT '单价',
	  `category` varchar(255) DEFAULT NULL COMMENT '所属类别',
	  `sort` varchar(4) DEFAULT NULL COMMENT '排序',
	  `create_time` datetime DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '创建时间',
	  `update_time` datetime DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
	  PRIMARY KEY (`id`) USING BTREE,
	  UNIQUE KEY `name` (`name`) USING BTREE
	) ENGINE=InnoDB AUTO_INCREMENT=819 DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC;
假设，我们现在需要将数据库中的`name`和`category`字段分别映射为`key`和`value`。本案例是以非web项目来演示。





# mybatis配置文件 #
`SqlMapConfig.xml`文件内容：

	<?xml version="1.0" encoding="UTF-8" ?>  
	<!DOCTYPE configuration PUBLIC "-//ibatis.apache.org//DTD Config 3.0//EN" 
		"http://ibatis.apache.org/dtd/ibatis-3-config.dtd">
	<configuration>
		<environments default="environment">
			<environment id="environment">
				<transactionManager type="JDBC" />
				<dataSource type="POOLED">
					<property name="driver" value="com.mysql.jdbc.Driver" />
					<property name="url" value="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false" />
					<property name="username" value="root"/>
					<property name="password" value="123123"/>
				</dataSource>
			</environment>
		</environments>
		<mappers>
			<mapper resource="mappers/FoodMapper.xml" />
		</mappers>
	</configuration>


# mybatis的xml文件 #
`FoodMapper.xml`文件内容：

	<?xml version="1.0" encoding="UTF-8" ?>  
	<!DOCTYPE mapper PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"      
	 "http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
	<mapper namespace="com.test.dao.FoodDao">
		<resultMap id="name_category" type="java.util.HashMap">
			<result property="key" column="name" />
			<result property="value" column="category" />
		</resultMap>
		
		<select id="findNameCategoryMap" parameterType="java.util.Map" resultMap="name_category">
			select name,category from food_new
		</select>
		
	</mapper>

# mybatis的工具类-与数据库建立连接 #

	package com.test.utils;
	
	import org.apache.ibatis.session.SqlSession;
	import org.apache.ibatis.session.SqlSessionFactory;
	import org.apache.ibatis.session.SqlSessionFactoryBuilder;
	
	public class MyBatisUtil {
		private static SqlSessionFactory sf;
	
		static {
			SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
			sf = builder.build(MyBatisUtil.class.getClassLoader().getResourceAsStream("SqlMapConfig.xml"));
		}
	
		public static SqlSession getSession() {
			return sf.openSession();
		}
	
		public static void main(String[] args) {
			SqlSession session = MyBatisUtil.getSession();
			System.out.println(session);
			session.close();
		}
	}

# dao #

	package com.test.dao;
	
	import java.util.Map;
	
	public interface FoodDao {
		
		Map<String, String> findNameCategoryMap();
		
}

# 自定义查找结果集--实现ResultHandler接口 #
`ResultHandler`,顾名思义,对返回的结果进行处理,最终得到自己想要的数据格式或类型。也就是说,可以自定义返回类型。

	package com.test.config;
	
	import java.util.HashMap;
	import java.util.Map;
	
	import org.apache.ibatis.session.ResultContext;
	import org.apache.ibatis.session.ResultHandler;
	
	public class NameCategoryMapResultHandler implements ResultHandler {
	
		private final Map<String, String> name_Category_Map = new HashMap<String, String>();
	
		@Override
		public void handleResult(ResultContext context) {
			@SuppressWarnings("unchecked")
			Map<String, String> map = (Map<String, String>) context.getResultObject();
			name_Category_Map.put(map.get("key"), map.get("value"));
		}
	
		public Map<String, String> getMappedResults() {
			return name_Category_Map;
		}
	
	}

# service #

	package com.test.service;
	
	import java.util.Map;
	
	public interface FoodService {
		
		Map<String, String> findNameCategoryMap();
	}

实现：

	package com.test.service.impl;
	
	import java.util.Map;
	
	import org.apache.ibatis.session.SqlSession;
	
	import com.zhuoshan.config.NameCategoryMapResultHandler;
	import com.zhuoshan.config.NameSortMapResultHandler;
	import com.zhuoshan.service.FoodService;
	import com.zhuoshan.utils.MyBatisUtil;
	
	public class FoodServiceImpl implements FoodService {
		
		private SqlSession session = MyBatisUtil.getSession();
	
		@Override
		public Map<String, String> findNameCategoryMap() {
			NameCategoryMapResultHandler handler = new NameCategoryMapResultHandler();
			session.select("com.test.dao.FoodDao.findNameCategoryMap", handler);
		    return handler.getMappedResults();
		}
		
		public static void main(String[] args) {
			FoodServiceImpl f = new FoodServiceImpl();
			System.out.println(f.findNameCategoryMap());
			System.out.println(f.findNameCategoryMap().size());
		}
	}

# 调用 #

	package com.test.core;
	
	import java.util.Map;
	
	import com.zhuoshan.service.FoodService;
	import com.zhuoshan.service.impl.FoodServiceImpl;
	
	public class FoodMain {
		public static void main(String[] args) throws Exception {
			FoodService foodService = new FoodServiceImpl();
			Map<String, String> name_category_map = foodService.findNameCategoryMap();
			name_category_map.forEach((key, value) -> System.out.println(key + ":" + value));
		}
	}