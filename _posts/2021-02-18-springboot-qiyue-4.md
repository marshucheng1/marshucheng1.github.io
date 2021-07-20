---
layout: post
title: Springboot-JPA基础内容
date: 2021-02-18 12:10:22
categories: springboot
tags: springboot
author: MarsHu
---

* content
{:toc}

### 多环境配置文件(profiles)以及启用方式  ###
假设在resources目录下新建一个配置文件夹config。在config文件夹下新建2个配置类。application-dev.yml和application-prod.yml。

这里需要注意的时,配置文件可以放在resources目录下任意位置,但是命名的规范必须是以application-开头。

application-dev.yml内容如下:

	server:
	  port: 8081

application-prod.yml内容如下:

	server:
	  port: 8080

将公共的环境配置放在application.yml中,并且配置需要运行dev还是prod配置,内容如下:

	spring:
	  profiles:
		#读取dev配置,如果需要读取prod配置,则将dev改成prod
	    active: dev
	
	zhqx:
	  api-package: com.zhqx.missyou.api

idea中新建的.yml文件没有提示解决

	1.点击File->2.点击Project Structure->3.点击Facets->4.点击中间栏目的Spring(项目名)->5.点击右边栏的小树叶(Customize spring boot)






### 引入jpa配置,使用实体生成数据库表  ###
数据库相关jar包

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

实体类Banner,相关基于注解的基本配置

	package com.zhqx.missyou.model;
	
	import javax.persistence.*;
	
	@Entity
	@Table(name = "xm_banner")//指定表名,可以不使用JPA默认表名规则,不使用该注解默认表名为banner
	@Where(clause = "delete_time is null")//指定查询条件
	public class Banner {
	    @Id//指定主键
		@GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    @Column(length = 16)//设置字段长度
	    private String name;
	    @Transient//不生成该名称对应的表字段
	    private String description;
	    private String img;
	    private String title;
	}

jpa相关配置application.yml:

	spring:
	  profiles:
	    active: dev
	  jpa:
	    hibernate:
	      ddl-auto: create-drop #实际开发中建议使用update
	
	zhqx:
	  api-package: com.zhqx.missyou.api


### 实体与实体之间一对多关系配置  ###

新建一个实体类BannerItem

	package com.zhqx.missyou.model;
	
	import javax.persistence.*;
	
	@Entity
	public class BannerItem {
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    private String img;
	    private String keyword;
	    private Short type;
	    private String name;
	}

一个Banner可以对应多个BannerItem.在Banner中添加BannerItem集合属性

	package com.zhqx.missyou.model;
	
	import javax.persistence.*;
	import java.util.List;
	
	@Entity
	public class Banner {
	    @Id//指定主键
		@GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    @Column(length = 16)//设置字段长度
	    private String name;
	    @Transient//不生成该名称对应的表字段
	    private String description;
	    private String img;
	    private String title;
	    
	    @OneToMany
	    private List<BannerItem> items;
	}

此时,运行程序,程序会自动生成三张表。banner、banner_item、banner_items。其中banner_items表中有2个字段.banner_id和items_id。
这个关系是由JPA默认建表规则生成。

因为在当前的banner_item表中没有字段表示属于哪一个banner,所有JPA才会自动生成第三张表来维护之间的关系。为了避免自动生成第三张表。
我们需要在BannerItem中新增一个字段bannerId表示外键关系。

	package com.zhqx.missyou.model;
	
	import javax.persistence.*;
	
	@Entity
	public class BannerItem {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    private String img;
	    private String keyword;
	    private Short type;
	    private String name;
		//这里JPA默认创建的表字段名为banner_id
	    private Long bannerId;
	}

同时我们需要在Banner中指定要关联的外键字段

	package com.zhqx.missyou.model;
	
	import javax.persistence.*;
	import java.util.List;
	
	@Entity
	public class Banner {
	    @Id//指定主键
	    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    @Column(length = 16)//设置字段长度
	    private String name;
	    @Transient//不生成该名称对应的表字段
	    private String description;
	    private String img;
	    private String title;
	
	    @OneToMany
	    @JoinColumn(name = "bannerId")
	    private List<BannerItem> items;
	}

此时,运行程序,就不会创建第三张表了,需要注意的是,此时banner表和banner_item表是存在物理外键的。

### JPA的Repository定义  ###
定义JPA查询接口

	package com.zhqx.missyou.repository;
	
	import com.zhqx.missyou.model.Banner;
	import org.springframework.data.jpa.repository.JpaRepository;
	import org.springframework.stereotype.Repository;
	
	@Repository
	public interface BannerRepository extends JpaRepository<Banner, Long> {
	    //JpaRepository<T, ID> T表示业务对象,ID表示主键的类型
	    
	    Banner findOneById(Long id);
	    Banner findOneByName(String name);
	}

在service中调用:

	package com.zhqx.missyou.service;
	
	import com.zhqx.missyou.model.Banner;
	
	public interface BannerService {
	        Banner getByName(String name);
	}
	==========================================
	package com.zhqx.missyou.service.impl;
	
	import com.zhqx.missyou.model.Banner;
	import com.zhqx.missyou.repository.BannerRepository;
	import com.zhqx.missyou.service.BannerService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Service;
	
	@Service
	public class BannerServiceImpl implements BannerService {
	
	    @Autowired
	    private BannerRepository bannerRepository;
	
	    @Override
	    public Banner getByName(String name) {
	        return bannerRepository.findOneByName(name);
	    }
	}

在控制器中调用:
	
	package com.zhqx.missyou.api.v1;
	
	import com.zhqx.missyou.model.Banner;
	import com.zhqx.missyou.service.BannerService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.validation.annotation.Validated;
	        import org.springframework.web.bind.annotation.*;
	
	        import javax.validation.constraints.NotBlank;
	
	@RestController
	@RequestMapping("/banner")
	@Validated
	public class BannerController {
	
	    @Autowired
	    private BannerService bannerService;
	
	    @GetMapping("/name/{name}")
	    public Banner getByName(@PathVariable @NotBlank String name) {
	        Banner banner = bannerService.getByName(name);
	        return banner;
	    }
	}

这里需要注意的是不要忘了补全Banner和BannerItem实体中的getter和setter方法,否则会出现返回结果为null的问题。因为使用了lombok插件。
所以分别在两个实体上加上@Getter和@Setter注解

打印SQL,并且格式化输出SQL配置,application-dev.yml中配置

	server:
	  port: 8080
	spring:
	  datasource:
	    url: jdbc:mysql://localhost:3306/missyou?characterEncoding=utf-8&serverTimezone=GMT%2B8
	    username: root
	    password: root
	  jpa:
	    show-sql: true
	    properties:
	      hibernate:
	        format_sql: true

在默认情况下JPA的一对多查询是一种惰性查询模式(懒加载模式),如果没有用到items属性时,是不会执行关联查询的。当然我们也可以要求在初始时,
就默认查出关联的items。使用注解@OneToMany(fetch = FetchType.EAGER)

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.*;
	import java.util.List;
	
	@Getter
	@Setter
	@Entity
	@Table(name = "banner")//指定表名
	public class Banner {
	    @Id//指定主键
	    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    @Column(length = 16)//设置字段长度
	    private String name;
	    @Transient//不生成该名称对应的表字段
	    private String description;
	    private String img;
	    private String title;
	
	    @OneToMany(fetch = FetchType.EAGER)//初始时,就关联查询出数据
	    @JoinColumn(name = "bannerId")
	    private List<BannerItem> items;
	}

如果需要测试这种情况,只需要将控制器中返回结果改为void,然后测试使用该注解和不使用该注解时,观察控制台打印内容就可以了。当然正常情况下,我们还是不要修改默认的惰性查询机制。


### 双向一对多配置  ###
在当前的配置中,我们可以通过banner查询出关联的bannerItem,如果我想通过bannerItem查询出关联的banner应该怎么做呢,这个时候就需要用到
双向一对多的配置。

修改BannerItem,增加关联属性banner,同时将原来的@JoinColumn注解使用在当前属性上

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.*;
	
	@Getter
	@Setter
	@Entity
	public class BannerItem {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    private String img;
	    private String keyword;
	    private Short type;
	    private String name;
	
	    private Long bannerId;
	
	    @ManyToOne
	    @JoinColumn(name = "bannerId")
	    private Banner banner;
	}

修改Banner,指定双向绑定的属性是什么,此案例中是banner,同时删除注解@JoinColumn

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.*;
	import java.util.List;
	
	@Getter
	@Setter
	@Entity
	@Table(name = "banner")//指定表名
	public class Banner {
	    @Id//指定主键
	    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    @Column(length = 16)//设置字段长度
	    private String name;
	    @Transient//不生成该名称对应的表字段
	    private String description;
	    private String img;
	    private String title;
	
	    @OneToMany(mappedBy = "banner", fetch = FetchType.EAGER)
	    private List<BannerItem> items;
	}

双向一对多的配置后,原来的bannerId就没有必要存在了,JPA规则中配置了双向一对多后,会自动生成banner_item,并生成外键。运行程序会出错,
需要我们将BannerItem中的bannerId属性给删除。

但是在实际开发过程中,我们有的时候是需要这个值的,所以我们需要进一步配置.

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.*;
	
	@Getter
	@Setter
	@Entity
	public class BannerItem {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    private String img;
	    private String keyword;
	    private Short type;
	    private String name;
	
	    private Long bannerId;
	
	    @ManyToOne
	    @JoinColumn(insertable = false, updatable = false, name = "bannerId")
	    private Banner banner;
	}

JPA默认双向一对多会自动生成外键物理字段banner_id,不利于值的获取,实际开发过程中可以选择性的使用单向一对多的配置。或者使用双向一对多时,使用上面的配置来解决。


### 多对多关系配置@ManyToMany  ###

#### 1.单向多对多关系配置  ####
Theme类:

	package com.zhqx.missyou.model;
	
	import javax.persistence.Entity;
	import javax.persistence.ManyToMany;
	import java.util.List;
	
	@Entity
	public class Theme {
		@Id
	    private Long id;
	    private String title;
	    private String name;
	
	    @ManyToMany
	    private List<Spu> spuList;
	}

Spu类:

	package com.zhqx.missyou.model;
	
	import javax.persistence.Entity;
	import javax.persistence.Id;
	
	@Entity
	public class Spu {
	    @Id
	    private Long id;
	    private String title;
	    private String subtitle;
	
	}

此时,JPA默认生成三张表:spu、theme、theme_spu_list。首先默认的情况下我们希望生成theme_spu表而不是后面加上一个list。而且在默认生成的theme_spu_list表中维护的外键字段是spu_list_id,实际上我们只需要生成spu_id这样的字段。

所以我们需要指定第三张表的名称以及相关外键字段名称。

	package com.zhqx.missyou.model;
	
	import javax.persistence.*;
	import java.util.List;
	
	@Entity
	public class Theme {
	    @Id
	    private Long id;
	    private String title;
	    private String name;
	
	    @ManyToMany
	    @JoinTable(name = "theme_spu", joinColumns = @JoinColumn(name = "theme_id"),
	    inverseJoinColumns = @JoinColumn(name = "spu_id"))
	    private List<Spu> spuList;
	}

#### 2.双向多对多关系配置  ####
在一对多关系中,一方表示被维护端,多方表示维护端。双向多对多关系,我们需要确定一个双向关系的维护端,我们让Spu成为关系的被维护端。

修改Spu类,指定多对多的属性(该属性对应Theme类的spuList属性):

	package com.zhqx.missyou.model;
	
	import javax.persistence.Entity;
	import javax.persistence.Id;
	import javax.persistence.ManyToMany;
	import java.util.List;
	
	@Entity
	public class Spu {
	    @Id
	    private Long id;
	    private String title;
	    private String subtitle;
	    
	    @ManyToMany(mappedBy = "spuList")
	    private List<Theme> themeList;
	}

### 禁止JPA创建物理外键  ###
1.在Banner类中使用注解@org.hibernate.annotations.ForeignKey

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.*;
	import java.util.List;
	
	@Getter
	@Setter
	@Entity
	@Table(name = "banner")//指定表名
	public class Banner {
	    @Id//指定主键
	    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    @Column(length = 16)//设置字段长度
	    private String name;
	    @Transient//不生成该名称对应的表字段
	    private String description;
	    private String img;
	    private String title;
	
	    @OneToMany(mappedBy = "banner", fetch = FetchType.EAGER)
	    @org.hibernate.annotations.ForeignKey(name = "null")
	    private List<BannerItem> items;
	}

2.在BannerItem配置,有BUG

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.*;
	
	@Getter
	@Setter
	@Entity
	public class BannerItem {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增长
	    private Long id;
	    private String img;
	    private String keyword;
	    private Short type;
	    private String name;
	
	    private Long bannerId;
	
	    @ManyToOne
	    @JoinColumn(foreignKey = @ForeignKey(value = ConstraintMode.NO_CONSTRAINT),insertable = false, updatable = false, name = "bannerId")
	    private Banner banner;
	}

### 使用idea逆向生成Entity  ###
使用的idea版本,2018.3.6.依次点击View->Tool Windows->Database

在弹出的Database窗口,点击+号->Data Source->Mysql.

在弹出的DataSources And Drivers窗口中,填写Database、user、password、host等信息。然后点击Apply,再点击OK,这有右边就会出现数据库连接窗口。

点击File->Project Structure->Modules->点击+号->选择JPA。

将Default JPA Provider修改为Hibernate,点击Apply,再点击OK。

View->Tool Windows->Persistence。在弹出的窗口中,点击missyou,右键->Generate Persistence Mapping->By DataBase Schema

在弹出的Import Database Schema窗口中依次设置Choose Data Source、以及需要将生成的Entity放在哪个Package。本案例中选择:com.zhqx.missyou.model

将Entity suffix里的后缀删除。

勾选需要生成的表,这里以banner表为例,然后点击OK,点击YES.生成的实体如下:

	package com.zhqx.missyou.model;
	
	import javax.persistence.Basic;
	import javax.persistence.Column;
	import javax.persistence.Entity;
	import javax.persistence.Id;
	import java.sql.Timestamp;
	import java.util.Objects;
	
	@Entity
	public class Banner {
	    private int id;
	    private String name;
	    private String description;
	    private Timestamp createTime;
	    private Timestamp updateTime;
	    private Timestamp deleteTime;
	    private String title;
	    private String img;
	
	    @Id
	    @Column(name = "id")
	    public int getId() {
	        return id;
	    }
	
	    public void setId(int id) {
	        this.id = id;
	    }
	
	    @Basic
	    @Column(name = "name")
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    @Basic
	    @Column(name = "description")
	    public String getDescription() {
	        return description;
	    }
	
	    public void setDescription(String description) {
	        this.description = description;
	    }
	
	    @Basic
	    @Column(name = "create_time")
	    public Timestamp getCreateTime() {
	        return createTime;
	    }
	
	    public void setCreateTime(Timestamp createTime) {
	        this.createTime = createTime;
	    }
	
	    @Basic
	    @Column(name = "update_time")
	    public Timestamp getUpdateTime() {
	        return updateTime;
	    }
	
	    public void setUpdateTime(Timestamp updateTime) {
	        this.updateTime = updateTime;
	    }
	
	    @Basic
	    @Column(name = "delete_time")
	    public Timestamp getDeleteTime() {
	        return deleteTime;
	    }
	
	    public void setDeleteTime(Timestamp deleteTime) {
	        this.deleteTime = deleteTime;
	    }
	
	    @Basic
	    @Column(name = "title")
	    public String getTitle() {
	        return title;
	    }
	
	    public void setTitle(String title) {
	        this.title = title;
	    }
	
	    @Basic
	    @Column(name = "img")
	    public String getImg() {
	        return img;
	    }
	
	    public void setImg(String img) {
	        this.img = img;
	    }
	
	    @Override
	    public boolean equals(Object o) {
	        if (this == o) return true;
	        if (o == null || getClass() != o.getClass()) return false;
	        Banner banner = (Banner) o;
	        return id == banner.id &&
	                Objects.equals(name, banner.name) &&
	                Objects.equals(description, banner.description) &&
	                Objects.equals(createTime, banner.createTime) &&
	                Objects.equals(updateTime, banner.updateTime) &&
	                Objects.equals(deleteTime, banner.deleteTime) &&
	                Objects.equals(title, banner.title) &&
	                Objects.equals(img, banner.img);
	    }
	
	    @Override
	    public int hashCode() {
	        return Objects.hash(id, name, description, createTime, updateTime, deleteTime, title, img);
	    }
	}

实际上默认生成的实体,有时候可能不太符合开发习惯.我们可以进行修改一下。调整后的Banner、BannerItem、BaseEntity数据如下:

Banner类:

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.*;
	import java.util.List;
	
	@Getter
	@Setter
	@Entity
	public class Banner extends BaseEntity {
	    @Id
	    private Long id;
	    private String name;
	    private String description;
	    private String title;
	    private String img;
	
	    @OneToMany(fetch = FetchType.LAZY)
	    @JoinColumn(name = "bannerId")
	    private List<BannerItem> items;
	}

BannerItem类:

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.*;
	
	@Getter
	@Setter
	@Entity
	public class BannerItem extends BaseEntity{
	    @Id
	    private Long id;
	    private String img;
	    private String keyword;
	    private short type;
	    private Long bannerId;
	    private String name;
	
	}

BaseEntity类:

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import java.util.Date;
	
	@Getter
	@Setter
	public abstract class BaseEntity {
	    private Date createTime;
	    private Date updateTime;
	    private Date deleteTime;
	}
