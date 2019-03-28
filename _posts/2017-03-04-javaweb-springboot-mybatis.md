---
layout: post
title: Springboot-mybatis使用
date: 2017-03-04 13:31:22
categories: JavaWeb
tags: springboot
author: MarsHu
---

* content
{:toc}

### SpringBoot配置支持mybatis ###

使用`@MapperScan`注解

	package com.hyhb;
	
	import org.mybatis.spring.annotation.MapperScan;
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	
	@SpringBootApplication
	@MapperScan(basePackages = "com.hyhb.dao")
	public class HyhbApplication {
	
	    public static void main(String[] args) {
	        SpringApplication.run(HyhbApplication.class, args);
	    }
	
	}

### mybatis接口示例 ###

	package com.hyhb.dao;
	
	import com.hyhb.models.PotentialCustomer;
	import org.apache.ibatis.annotations.Param;
	
	import java.util.List;
	
	public interface PotentialCustomerDao {
	    PotentialCustomer getById(Integer id) throws Exception;
	
	    PotentialCustomer getByContactNumber(String contactNumber) throws Exception;
	
	    List<PotentialCustomer> getListByParams(@Param("ownerList") List<String> ownerList, @Param("type") String type, @Param("area") String area, @Param("statusList") List<String> statusList, @Param("contactNumber") String contactNumber, @Param("isActive") Integer isActive, @Param("page") Integer page, @Param("pageSize") Integer pageSize) throws Exception;
	
	    Integer getCountByParams(@Param("ownerList") List<String> ownerList, @Param("type") String type, @Param("area") String area, @Param("statusList") List<String> statusList, @Param("contactNumber") String contactNumber, @Param("isActive") Integer isActive) throws Exception;
	
	    Integer saveOrUpdate(PotentialCustomer potentialCustomer) throws Exception;
	
	    Integer delete(PotentialCustomer potentialCustomer) throws Exception;
	}

### mybatis接口对应Mapper文件示例 ###

字段有点多，可以直接忽略，其中`saveOrUpdate`方法可以实现和`hibernate`一样的效果。一条SQL搞定增加和修改。

	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
	<mapper namespace="com.hyhb.dao.PotentialCustomerDao">
	
	    <select id="getById" parameterType="java.lang.Integer" resultType="com.hyhb.models.PotentialCustomer">
	        SELECT * FROM potential_customer WHERE id=#{id}
	    </select>
	
	    <select id="getByContactNumber" parameterType="java.lang.String" resultType="com.hyhb.models.PotentialCustomer">
	        SELECT * FROM potential_customer WHERE
	        contactNumber1=#{contactNumber} OR
	        contactNumber2=#{contactNumber} OR
	        contactNumber3=#{contactNumber} OR
	        contactNumber4=#{contactNumber} OR
	        contactNumber5=#{contactNumber}
	    </select>
	
	    <select id="getListByParams" resultType="com.hyhb.models.PotentialCustomer">
	        SELECT * FROM potential_customer WHERE 1=1
	        <if test="ownerList != null">
	            AND owner in
	            <foreach collection="ownerList" index="index" item="owner" open="(" separator="," close=")">
	                #{owner}
	            </foreach>
	        </if>
	        <if test="type != null">
	            AND type=#{type}
	        </if>
	        <if test="area != null">
	            AND area=#{area}
	        </if>
	        <if test="statusList != null">
	            AND salesStatus in
	            <foreach collection="statusList" index="index" item="status" open="(" separator="," close=")">
	                #{status}
	            </foreach>
	        </if>
	        <if test="contactNumber != null">
	            AND contactNumber1=#{contactNumber} OR
	            contactNumber2=#{contactNumber} OR
	            contactNumber3=#{contactNumber} OR
	            contactNumber4=#{contactNumber} OR
	            contactNumber5=#{contactNumber}
	        </if>
	        <if test="isActive != null">
	            AND isActive=#{isActive}
	        </if>
	        ORDER BY createTime DESC
	        <if test="page != null and pageSize != null">
	            limit ${(page-1)*pageSize},${pageSize}
	        </if>
	    </select>
	
	    <select id="getCountByParams" resultType="java.lang.Integer">
	        SELECT COUNT(id) FROM potential_customer WHERE 1=1
	        <if test="ownerList != null">
	            AND owner in
	            <foreach collection="ownerList" index="index" item="owner" open="(" separator="," close=")">
	                #{owner}
	            </foreach>
	        </if>
	        <if test="type != null">
	            AND type=#{type}
	        </if>
	        <if test="area != null">
	            AND area=#{area}
	        </if>
	        <if test="statusList != null">
	            AND salesStatus in
	            <foreach collection="statusList" index="index" item="status" open="(" separator="," close=")">
	                #{status}
	            </foreach>
	        </if>
	        <if test="contactNumber != null">
	            AND contactNumber1=#{contactNumber} OR
	            contactNumber2=#{contactNumber} OR
	            contactNumber3=#{contactNumber} OR
	            contactNumber4=#{contactNumber} OR
	            contactNumber5=#{contactNumber}
	        </if>
	        <if test="isActive != null">
	            AND isActive=#{isActive}
	        </if>
	    </select>
	
	    <insert id="saveOrUpdate" parameterType="com.hyhb.models.PotentialCustomer">
	        INSERT INTO potential_customer
	        <trim prefix="(" suffix=")" suffixOverrides=",">
	            <if test="id != null">
	                `id`,
	            </if>
	            <if test="createTime != null">
	                `createTime`,
	            </if>
	            <if test="owner != null">
	                `owner`,
	            </if>
	            <if test="type != null">
	                `type`,
	            </if>
	            <if test="visitTime != null">
	                `visitTime`,
	            </if>
	            <if test="company != null">
	                `company`,
	            </if>
	            <if test="area != null">
	                `area`,
	            </if>
	            <if test="alreadyQualified != null">
	                `alreadyQualified`,
	            </if>
	            <if test="visitProject != null">
	                `visitProject`,
	            </if>
	            <if test="personnelSituation != null">
	                `personnelSituation`,
	            </if>
	            <if test="messageFrom != null">
	                `messageFrom`,
	            </if>
	            <if test="revisitTime != null">
	                `revisitTime`,
	            </if>
	            <if test="revisitMark != null">
	                `revisitMark`,
	            </if>
	            <if test="remindTime != null">
	                `remindTime`,
	            </if>
	            <if test="visitingTime != null">
	                `visitingTime`,
	            </if>
	            <if test="visitingMark != null">
	                `visitingMark`,
	            </if>
	            <if test="username != null">
	                `username`,
	            </if>
	            <if test="contactNumber1 != null">
	                `contactNumber1`,
	            </if>
	            <if test="contactNumber2 != null">
	                `contactNumber2`,
	            </if>
	            <if test="contactNumber3 != null">
	                `contactNumber3`,
	            </if>
	            <if test="contactNumber4 != null">
	                `contactNumber4`,
	            </if>
	            <if test="contactNumber5 != null">
	                `contactNumber5`,
	            </if>
	            <if test="name1 != null">
	                `name1`,
	            </if>
	            <if test="name2 != null">
	                `name2`,
	            </if>
	            <if test="name3 != null">
	                `name3`,
	            </if>
	            <if test="name4 != null">
	                `name4`,
	            </if>
	            <if test="name5 != null">
	                `name5`,
	            </if>
	            <if test="qqNo != null">
	                `qqNo`,
	            </if>
	            <if test="wxNo != null">
	                `wxNo`,
	            </if>
	            <if test="mark != null">
	                `mark`,
	            </if>
	            <if test="salesStatus != null">
	                `salesStatus`,
	            </if>
	            <if test="revisitCount != null">
	                `revisitCount`,
	            </if>
	            <if test="visitingCount != null">
	                `visitingCount`,
	            </if>
	            <if test="isActive != null">
	                `isActive`,
	            </if>
	            <if test="updateTime != null">
	                `updateTime`
	            </if>
	        </trim>
	        <trim prefix="VALUES(" suffix=")" suffixOverrides=",">
	            <if test="id != null">
	                #{id,jdbcType=INTEGER},
	            </if>
	            <if test="createTime != null">
	                #{createTime,jdbcType=TIMESTAMP},
	            </if>
	            <if test="owner != null">
	                #{owner,jdbcType=INTEGER},
	            </if>
	            <if test="type != null">
	                #{type,jdbcType=VARCHAR},
	            </if>
	            <if test="visitTime != null">
	                #{visitTime,jdbcType=TIMESTAMP},
	            </if>
	            <if test="company != null">
	                #{company,jdbcType=VARCHAR},
	            </if>
	            <if test="area != null">
	                #{area,jdbcType=VARCHAR},
	            </if>
	            <if test="alreadyQualified != null">
	                #{alreadyQualified,jdbcType=VARCHAR},
	            </if>
	            <if test="visitProject != null">
	                #{visitProject,jdbcType=VARCHAR},
	            </if>
	            <if test="personnelSituation != null">
	                #{personnelSituation,jdbcType=VARCHAR},
	            </if>
	            <if test="messageFrom != null">
	                #{messageFrom,jdbcType=VARCHAR},
	            </if>
	            <if test="revisitTime != null">
	                #{revisitTime,jdbcType=TIMESTAMP},
	            </if>
	            <if test="revisitMark != null">
	                #{revisitMark,jdbcType=LONGVARCHAR},
	            </if>
	            <if test="remindTime != null">
	                #{remindTime,jdbcType=TIMESTAMP},
	            </if>
	            <if test="visitingTime != null">
	                #{visitingTime,jdbcType=TIMESTAMP},
	            </if>
	            <if test="visitingMark != null">
	                #{visitingMark,jdbcType=LONGVARCHAR},
	            </if>
	            <if test="username != null">
	                #{username,jdbcType=VARCHAR},
	            </if>
	            <if test="contactNumber1 != null">
	                #{contactNumber1,jdbcType=VARCHAR},
	            </if>
	            <if test="contactNumber2 != null">
	                #{contactNumber2,jdbcType=VARCHAR},
	            </if>
	            <if test="contactNumber3 != null">
	                #{contactNumber3,jdbcType=VARCHAR},
	            </if>
	            <if test="contactNumber4 != null">
	                #{contactNumber4,jdbcType=VARCHAR},
	            </if>
	            <if test="contactNumber5 != null">
	                #{contactNumber5,jdbcType=VARCHAR},
	            </if>
	            <if test="name1 != null">
	                #{name1,jdbcType=VARCHAR},
	            </if>
	            <if test="name2 != null">
	                #{name2,jdbcType=VARCHAR},
	            </if>
	            <if test="name3 != null">
	                #{name3,jdbcType=VARCHAR},
	            </if>
	            <if test="name4 != null">
	                #{name4,jdbcType=VARCHAR},
	            </if>
	            <if test="name5 != null">
	                #{name5,jdbcType=VARCHAR},
	            </if>
	            <if test="mark != null">
	                #{mark,jdbcType=LONGVARCHAR},
	            </if>
	            <if test="qqNo != null">
	                #{qqNo,jdbcType=VARCHAR},
	            </if>
	            <if test="wxNo != null">
	                #{wxNo,jdbcType=VARCHAR},
	            </if>
	            <if test="salesStatus != null">
	                #{salesStatus,jdbcType=VARCHAR},
	            </if>
	            <if test="revisitCount != null">
	                #{revisitCount,jdbcType=INTEGER},
	            </if>
	            <if test="visitingCount != null">
	                #{visitingCount,jdbcType=INTEGER},
	            </if>
	            <if test="isActive != null">
	                #{isActive,jdbcType=INTEGER},
	            </if>
	            <if test="updateTime != null">
	                #{updateTime,jdbcType=TIMESTAMP}
	            </if>
	        </trim>
	        ON DUPLICATE KEY UPDATE
	        <trim suffixOverrides=",">
	            <if test="createTime != null">
	                `createTime`=#{createTime,jdbcType=TIMESTAMP},
	            </if>
	            <if test="owner != null">
	                `owner`=#{owner,jdbcType=INTEGER},
	            </if>
	            <if test="type != null">
	                `type`=#{type,jdbcType=VARCHAR},
	            </if>
	            <if test="visitTime != null">
	                `visitTime`=#{visitTime,jdbcType=TIMESTAMP},
	            </if>
	            <if test="company != null">
	                `company`=#{company,jdbcType=VARCHAR},
	            </if>
	            <if test="area != null">
	                `area`=#{area,jdbcType=VARCHAR},
	            </if>
	            <if test="alreadyQualified != null">
	                `alreadyQualified`=#{alreadyQualified,jdbcType=VARCHAR},
	            </if>
	            <if test="visitProject != null">
	                `visitProject`=#{visitProject,jdbcType=VARCHAR},
	            </if>
	            <if test="personnelSituation != null">
	                `personnelSituation`=#{personnelSituation,jdbcType=VARCHAR},
	            </if>
	            <if test="messageFrom != null">
	                `messageFrom`=#{messageFrom,jdbcType=VARCHAR},
	            </if>
	            <if test="revisitTime != null">
	                `revisitTime`=#{revisitTime,jdbcType=TIMESTAMP},
	            </if>
	            <if test="revisitMark != null">
	                `revisitMark`=#{revisitMark,jdbcType=LONGVARCHAR},
	            </if>
	            <if test="remindTime != null">
	                `remindTime`=#{remindTime,jdbcType=TIMESTAMP},
	            </if>
	            <if test="visitingTime != null">
	                `visitingTime`=#{visitingTime,jdbcType=TIMESTAMP},
	            </if>
	            <if test="visitingMark != null">
	                `visitingMark`=#{visitingMark,jdbcType=LONGVARCHAR},
	            </if>
	            <if test="username != null">
	                `username`=#{username,jdbcType=VARCHAR},
	            </if>
	            <if test="contactNumber1 != null">
	                `contactNumber1`=#{contactNumber1,jdbcType=VARCHAR},
	            </if>
	            <if test="contactNumber2 != null">
	                `contactNumber2`=#{contactNumber2,jdbcType=VARCHAR},
	            </if>
	            <if test="contactNumber3 != null">
	                `contactNumber3`=#{contactNumber3,jdbcType=VARCHAR},
	            </if>
	            <if test="contactNumber4 != null">
	                `contactNumber4`=#{contactNumber4,jdbcType=VARCHAR},
	            </if>
	            <if test="contactNumber5 != null">
	                `contactNumber5`=#{contactNumber5,jdbcType=VARCHAR},
	            </if>
	            <if test="name1 != null">
	                `name1`=#{name1,jdbcType=VARCHAR},
	            </if>
	            <if test="name2 != null">
	                `name2`=#{name2,jdbcType=VARCHAR},
	            </if>
	            <if test="name3 != null">
	                `name3`=#{name3,jdbcType=VARCHAR},
	            </if>
	            <if test="name4 != null">
	                `name4`=#{name4,jdbcType=VARCHAR},
	            </if>
	            <if test="name5 != null">
	                `name5`=#{name5,jdbcType=VARCHAR},
	            </if>
	            <if test="mark != null">
	                `mark`=#{mark,jdbcType=LONGVARCHAR},
	            </if>
	            <if test="qqNo != null">
	                `qqNo`=#{qqNo,jdbcType=VARCHAR},
	            </if>
	            <if test="wxNo != null">
	                `wxNo`=#{wxNo,jdbcType=VARCHAR},
	            </if>
	            <if test="salesStatus != null">
	                `salesStatus`=#{salesStatus,jdbcType=VARCHAR},
	            </if>
	            <if test="revisitCount != null">
	                `revisitCount`=#{revisitCount,jdbcType=INTEGER},
	            </if>
	            <if test="visitingCount != null">
	                `visitingCount`=#{visitingCount,jdbcType=INTEGER},
	            </if>
	            <if test="isActive != null">
	                `isActive`=#{isActive,jdbcType=INTEGER},
	            </if>
	            <if test="updateTime != null">
	                `updateTime`=#{updateTime,jdbcType=TIMESTAMP}
	            </if>
	        </trim>
	    </insert>
	
	    <delete id="delete" parameterType="com.hyhb.models.PotentialCustomer">
	        DELETE FROM potential_customer WHERE id=#{id}
	    </delete>
	
	</mapper>

### service层示例 ###

这里只有基本的业务操作。可以根据实际情况构建方法。

	package com.hyhb.service;
	
	import com.hyhb.models.PotentialCustomer;
	import com.hyhb.views.forms.dashboard.DashboardPotentialCustomerForm;
	
	import java.util.List;
	
	public interface PotentialCustomerService {
	    PotentialCustomer getById(Integer id) throws Exception;
	
	    PotentialCustomer getByContactNumber(String contactNumber) throws Exception;
	
	    List<PotentialCustomer> getListByParams(List<String> ownerList, String type, String area, List<String> statusList, String contactNumber, Integer isActive, Integer page, Integer pageSize) throws Exception;
	
	    Integer getCountByParams(List<String> ownerList, String type, String area, List<String> statusList, String contactNumber, Integer isActive) throws Exception;
	
	    PotentialCustomer saveOrUpdate(PotentialCustomer potentialCustomer) throws Exception;
	
	    Integer delete(PotentialCustomer potentialCustomer) throws Exception;
	}

### service实现层示例 ###

	package com.hyhb.service.impl;
	
	import com.hyhb.dao.PotentialCustomerDao;
	import com.hyhb.models.PotentialCustomer;
	import com.hyhb.service.PotentialCustomerService;
	import com.hyhb.utils.DateTimeUtil;
	import com.hyhb.views.forms.dashboard.DashboardPotentialCustomerForm;
	import org.springframework.stereotype.Service;
	
	import javax.annotation.Resource;
	import java.util.List;
	
	@Service
	public class PotentialCustomerServiceImpl implements PotentialCustomerService {
	
	    @Resource
	    private PotentialCustomerDao potentialCustomerDao;
	
	    @Override
	    public PotentialCustomer getById(Integer id) throws Exception {
	        return potentialCustomerDao.getById(id);
	    }
	
	    @Override
	    public PotentialCustomer getByContactNumber(String contactNumber) throws Exception {
	        return potentialCustomerDao.getByContactNumber(contactNumber);
	    }
	
	    @Override
	    public List<PotentialCustomer> getListByParams(List<String> ownerList, String type, String area, List<String> statusList, String contactNumber, Integer isActive, Integer page, Integer pageSize) throws Exception {
	        return potentialCustomerDao.getListByParams(ownerList, type, area, statusList, contactNumber, isActive, page, pageSize);
	    }
	
	    @Override
	    public Integer getCountByParams(List<String> ownerList, String type, String area, List<String> statusList, String contactNumber, Integer isActive) throws Exception {
	        return potentialCustomerDao.getCountByParams(ownerList, type, area, statusList, contactNumber, isActive);
	    }
	
	    @Override
	    public PotentialCustomer saveOrUpdate(PotentialCustomer potentialCustomer) throws Exception {
	        Integer result = potentialCustomerDao.saveOrUpdate(potentialCustomer);
	        return potentialCustomer;
	    }
	
	    @Override
	    public Integer delete(PotentialCustomer potentialCustomer) throws Exception {
	        return potentialCustomerDao.delete(potentialCustomer);
	    }
	}
