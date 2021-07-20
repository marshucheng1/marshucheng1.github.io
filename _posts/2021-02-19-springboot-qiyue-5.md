---
layout: post
title: Springboot-数据的返回
date: 2021-02-19 12:10:22
categories: springboot
tags: springboot
author: MarsHu
---

* content
{:toc}

### Jaskson序列化库的重要配置  ###
通过前面的步骤,我们可以通过PostMan发送请求,拿回数据。通过访问:localhost:8080/v1/banner/name/b-1,拿回以下数据。

	{
	    "createTime": null,
	    "updateTime": null,
	    "deleteTime": null,
	    "id": 1,
	    "name": "b-1",
	    "description": "首页顶部主banner",
	    "title": null,
	    "img": "http://xxx.xxx.xxx/d87e180f-0f8a-480c-966a-f2f487b801cc.png",
	    "items": [
	        {
	            "createTime": null,
	            "updateTime": null,
	            "deleteTime": null,
	            "id": 12,
	            "img": "http://xxx.xxx.xxx/m1.png",
	            "keyword": "t-2",
	            "type": 3,
	            "bannerId": 1,
	            "name": null
	        },
	        {
	            "createTime": null,
	            "updateTime": null,
	            "deleteTime": null,
	            "id": 13,
	            "img": "http://xxx.xxx.xxx/assets/702f2ce9-5729-4aa4-aeb3-921513327747.pn",
	            "keyword": "23",
	            "type": 1,
	            "bannerId": 1,
	            "name": null
	        },
	        {
	            "createTime": null,
	            "updateTime": null,
	            "deleteTime": null,
	            "id": 14,
	            "img": "http://xxx.xxx.xxx/assets/b8e510a1-8340-43c2-a4b0-0e56a40256f9.png",
	            "keyword": "24",
	            "type": 1,
	            "bannerId": 1,
	            "name": null
	        }
	    ]
	}








springboot默认使用json库是jackson,观察返回结果,我们可以发现一下问题:首先数据库中createTime等time字段是有值的,但是前端返回数据中没有值.其次,数据库中createTime字段为create_time,而前端返回的数据字段为createTime。

关于createTime字段没有值的问题,是因为我们提取了积累BaseEntity.我们需要在基类上使用注解@MappedSuperclass

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.MappedSuperclass;
	import java.util.Date;
	
	@Getter
	@Setter
	@MappedSuperclass
	public abstract class BaseEntity {
	    private Date createTime;
	    private Date updateTime;
	    private Date deleteTime;
	}

再次请求时,就能得到日期字段的值了。如果需要将返回字段createTime改为create_time,我们可以修改application.yml文件

	spring:
	  profiles:
	    active: dev
	  jpa:
	    hibernate:
	      ddl-auto: none
	  jackson:
	    property-naming-strategy: SNAKE_CASE
	
	zhqx:
	  api-package: com.zhqx.missyou.api

再次请求时,createTime就变成了create_time。此时的日期数据如下:

	"create_time": "2019-07-27T20:47:15.000+00:00",

这个时候返回的日期格式为标准日期格式,假如我们只想返回时间戳日期格式,可在application.yml文件中配置如下:

	spring:
	  profiles:
	    active: dev
	  jpa:
	    hibernate:
	      ddl-auto: none
	  jackson:
	    property-naming-strategy: SNAKE_CASE
	    serialization:
	      WRITE_DATES_AS_TIMESTAMPS: true
	
	zhqx:
	  api-package: com.zhqx.missyou.api

再次请求时,就能得到时间戳格式的日期字段:

	"create_time": 1564260435000,

如果,我们不希望前端返回的json数据中有相关日期字段,我们可以进行如下设置:

	package com.zhqx.missyou.model;
	
	import com.fasterxml.jackson.annotation.JsonIgnore;
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.MappedSuperclass;
	import java.util.Date;
	
	@Getter
	@Setter
	@MappedSuperclass
	public abstract class BaseEntity {
	
	    @JsonIgnore
	    private Date createTime;
	    @JsonIgnore
	    private Date updateTime;
	    @JsonIgnore
	    private Date deleteTime;
	}


再次请求时,我们可以得到如下请求内容:

	{
	    "id": 1,
	    "name": "b-1",
	    "description": "首页顶部主banner",
	    "title": null,
	    "img": "http://xxx.xxx.xxx/d87e180f-0f8a-480c-966a-f2f487b801cc.png",
	    "items": [
	        {
	            "id": 12,
	            "img": "http://xxx.xxx.xxx/m1.png",
	            "keyword": "t-2",
	            "type": 3,
	            "banner_id": 1,
	            "name": null
	        },
	        {
	            "id": 13,
	            "img": "http://xxx.xxx.xxx/assets/702f2ce9-5729-4aa4-aeb3-921513327747.pn",
	            "keyword": "23",
	            "type": 1,
	            "banner_id": 1,
	            "name": null
	        },
	        {
	            "id": 14,
	            "img": "http://xxx.xxx.xxx/assets/b8e510a1-8340-43c2-a4b0-0e56a40256f9.png",
	            "keyword": "24",
	            "type": 1,
	            "banner_id": 1,
	            "name": null
	        }
	    ]
	}

### 对Banner资源进行null值判断  ###
BannerController类的修改

	package com.zhqx.missyou.api.v1;
	
	import com.zhqx.missyou.exception.http.NotFoundException;
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
	        if (banner == null) {
	            throw new NotFoundException(30005);
	        }
	        return banner;
	    }
	}

在exception-code.properties配置文件中新增错误信息配置。

	zhqx.codes[10000]= 通用异常
	zhqx.codes[10001]= 通用参数异常
	
	zhqx.codes[30005]= Banner类的资源不存在

### DozerBeanMapper拷贝属性 ###
传统的BeanUtils工具类拷贝不能够进行深拷贝,即假如目标对象中有对象类型数据时,使用该工具会有问题(未验证!)
引入相关Jar包

	<dependency>
        <groupId>com.github.dozermapper</groupId>
        <artifactId>dozer-core</artifactId>
        <version>6.5.0</version>
    </dependency>

Spu类:

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.Entity;
	import javax.persistence.Id;
	import javax.persistence.JoinColumn;
	import javax.persistence.OneToMany;
	import java.util.List;
	
	@Getter
	@Setter
	@Entity
	public class Spu extends BaseEntity {
	    @Id
	    private Long id;
	    private String title;
	    private String subtitle;
	    private Long categoryId;
	    private Long rootCategoryId;
	    private Boolean online;
	    private String price;
	    private Long sketchSpecId;
	    private Long defaultSkuId;
	    private String img;
	    private String discountPrice;
	    private String description;
	    private String tags;
	    private Boolean isTest;
	    //private Object spuThemeImg;
	    private String forThemeImg;
	
	    @OneToMany
	    @JoinColumn(name = "spuId")
	    private List<Sku> skuList;
	
	    @OneToMany
	    @JoinColumn(name = "spuId")
	    private List<SpuImg> spuImgList;
	
	    @OneToMany
	    @JoinColumn(name = "spuId")
	    private List<SpuDetailImg> spuDetailImgList;
	}

Sku类:

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.Entity;
	import javax.persistence.Id;
	import java.math.BigDecimal;
	
	@Getter
	@Setter
	@Entity
	public class Sku extends BaseEntity {
	    @Id
	    private Long id;
	    private BigDecimal price;
	    private BigDecimal discountPrice;
	    private Boolean online;
	    private String img;
	    private String title;
	    private Long spuId;
	
	    private String specs;
	    private String code;
	    private Long stock;
	    private Long categoryId;
	    private Long rootCategoryId;
	
	}

SpuDetailImg类:

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.*;
	
	@Getter
	@Setter
	@Entity
	public class SpuDetailImg extends BaseEntity {
	    @Id
	    private Long id;
	    private String img;
	    private Long spuId;
	    private Long index;
	
	}

SpuImg类:

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.*;
	import java.sql.Timestamp;
	import java.util.Objects;
	
	@Getter
	@Setter
	@Entity
	public class SpuImg extends BaseEntity {
	    @Id
	    private Long id;
	    private String img;
	    private Long spuId;
	
	}

SpuRepository类:

	package com.zhqx.missyou.repository;
	
	import com.zhqx.missyou.model.Spu;
	import org.springframework.data.jpa.repository.JpaRepository;
	import org.springframework.stereotype.Repository;
	
	@Repository
	public interface SpuRepository extends JpaRepository<Spu, Long> {
	    Spu findOneById(Long id);
	}

SpuServiceImpl类:

	package com.zhqx.missyou.service.impl;
	
	import com.zhqx.missyou.model.Spu;
	import com.zhqx.missyou.repository.SpuRepository;
	import com.zhqx.missyou.service.SpuService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Service;
	
	import java.util.List;
	
	@Service
	public class SpuServiceImpl implements SpuService {
	
	    @Autowired
	    private SpuRepository spuRepository;
	
	    @Override
	    public Spu getSpu(Long id) {
	        return spuRepository.findOneById(id);
	    }
	
	    @Override
	    public List<Spu> getLatestPagingSpu() {
	        return spuRepository.findAll();
	    }
	}

SpuController类:

	package com.zhqx.missyou.api.v1;
	
	import com.github.dozermapper.core.DozerBeanMapperBuilder;
	import com.github.dozermapper.core.Mapper;
	import com.zhqx.missyou.exception.http.NotFoundException;
	import com.zhqx.missyou.model.Spu;
	import com.zhqx.missyou.service.SpuService;
	import com.zhqx.missyou.vo.SpuSimplifyVO;
	import org.springframework.beans.BeanUtils;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.validation.annotation.Validated;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	import javax.validation.constraints.Positive;
	import java.util.ArrayList;
	import java.util.List;
	
	@RestController
	@RequestMapping("/spu")
	@Validated
	public class SpuController {
	
	    @Autowired
	    private SpuService spuService;
	
	    @GetMapping("/id/{id}/detail")//@Positive整数
	    public Spu getDetail(@PathVariable @Positive Long id) {
	        Spu spu = spuService.getSpu(id);
	        if(spu == null) {
	            throw new NotFoundException(30003);
	        }
	        return spu;
	    }
	
	    @GetMapping("/id/{id}/simplify")
	    public SpuSimplifyVO getSimplifyVO(@PathVariable @Positive Long id) {
	        Spu spu = spuService.getSpu(id);
	        SpuSimplifyVO vo = new SpuSimplifyVO();
	        BeanUtils.copyProperties(spu, vo);
	        return vo;
	    }
	
	    @GetMapping("/latest")
	    public List<SpuSimplifyVO> getLatestSpuList() {
	        Mapper mapper = DozerBeanMapperBuilder.buildDefault();
	        List<Spu> spuList = spuService.getLatestPagingSpu();
	        List<SpuSimplifyVO> vos = new ArrayList<>();
	        spuList.forEach(spu -> {
	            SpuSimplifyVO vo = mapper.map(spu, SpuSimplifyVO.class);
	            vos.add(vo);
	        });
	
	        return vos;
	    }
	}

### 使用Pageable分页,自定义分页结果集  ###
修改SpuServiceImpl方法,将分页查询结果返回封装到Page对象

	package com.zhqx.missyou.service.impl;
	
	import com.zhqx.missyou.model.Spu;
	import com.zhqx.missyou.repository.SpuRepository;
	import com.zhqx.missyou.service.SpuService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.domain.Page;
	import org.springframework.data.domain.PageRequest;
	import org.springframework.data.domain.Pageable;
	import org.springframework.data.domain.Sort;
	import org.springframework.stereotype.Service;
	
	@Service
	public class SpuServiceImpl implements SpuService {
	
	    @Autowired
	    private SpuRepository spuRepository;
	
	    @Override
	    public Spu getSpu(Long id) {
	        return spuRepository.findOneById(id);
	    }
	
	    @Override
	    public Page<Spu> getLatestPagingSpu(Integer pageNum, Integer size) {
	        //这里的排序字段对应的应该是实体中的属性,而不是数据库中的字段
	        Pageable pageable = PageRequest.of(pageNum, size, Sort.by("createTime").descending());
	        return spuRepository.findAll(pageable);
	    }
	}

定义分页实体PageCounter,包含page属性和count属性:

	package com.zhqx.missyou.bo;
	
	import lombok.Getter;
	import lombok.Setter;
	
	@Getter
	@Setter
	public class PageCounter {
	    private Integer page;
	    private Integer count;
	}

定义工具类,用来将前端传递的start参数和count参数封装到对象PageCounter中
	
	package com.zhqx.missyou.util;
	
	import com.zhqx.missyou.bo.PageCounter;
	
	public class CommonUtil {
	
	    //start:第几条数据开始查询,count:查询多少条数据
	    public static PageCounter convertToPageParameter(Integer start, Integer count) {
	        //当前页码
	        int pageNum = start/count;
	        PageCounter pageCounter = new PageCounter();
	        pageCounter.setPage(pageNum);
	        pageCounter.setCount(count);
	        return pageCounter;
	    }
	}

定义分页数据对象Paging,包含分页信息以及数据信息

	package com.zhqx.missyou.vo;
	
	import lombok.Getter;
	import lombok.NoArgsConstructor;
	import lombok.Setter;
	import org.springframework.data.domain.Page;
	
	import java.util.List;
	
	@Getter
	@Setter
	@NoArgsConstructor
	public class Paging<T> {
	    //总数
	    private Long total;
	    //页面显示数
	    private Integer count;
	    //当前页
	    private Integer page;
	    //总页数
	    private Integer totalPage;
	    private List<T> items;
	
	    public Paging(Page<T> pageT) {
	        this.initPageParameters(pageT);
	        this.items = pageT.getContent();
	    }
	
	    void initPageParameters(Page<T> pageT) {
	        this.total = pageT.getTotalElements();
	        this.count = pageT.getSize();
	        this.page = pageT.getNumber();
	        this.totalPage = pageT.getTotalPages();
	    }
	}

定义类PagingDozer,将原始的Page数据中返回的集合数据进行属性的舍弃,封装到新的返回数据对象中,将整个数据的拷贝放到此类中完成

	package com.zhqx.missyou.vo;
	
	import com.github.dozermapper.core.DozerBeanMapperBuilder;
	import com.github.dozermapper.core.Mapper;
	import org.springframework.data.domain.Page;
	
	import java.util.ArrayList;
	import java.util.List;
	
	//T:源数据类型;K:目标数据类型
	public class PagingDozer<T, K> extends Paging {
	
	    //kClass:目标数据类型
	    public PagingDozer(Page<T> pageT, Class<K> kClass) {
	        this.initPageParameters(pageT);
	
	        List<T> tList = pageT.getContent();
	        Mapper mapper = DozerBeanMapperBuilder.buildDefault();
	
	        List<K> voList = new ArrayList<>();
	
	        tList.forEach(t -> {
	            K vo = mapper.map(t, kClass);
	            voList.add(vo);
	        });
	        this.setItems(voList);
	    }
	}

修改SpuController类:

	package com.zhqx.missyou.api.v1;

	import com.zhqx.missyou.bo.PageCounter;
	import com.zhqx.missyou.exception.http.NotFoundException;
	import com.zhqx.missyou.model.Spu;
	import com.zhqx.missyou.service.SpuService;
	import com.zhqx.missyou.util.CommonUtil;
	import com.zhqx.missyou.vo.PagingDozer;
	import com.zhqx.missyou.vo.SpuSimplifyVO;
	import org.springframework.beans.BeanUtils;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.domain.Page;
	import org.springframework.validation.annotation.Validated;
	import org.springframework.web.bind.annotation.*;
	
	import javax.validation.constraints.Positive;
	
	@RestController
	@RequestMapping("/spu")
	@Validated
	public class SpuController {
	
	    @Autowired
	    private SpuService spuService;
	
	    @GetMapping("/id/{id}/detail")//@Positive整数
	    public Spu getDetail(@PathVariable @Positive Long id) {
	        Spu spu = spuService.getSpu(id);
	        if(spu == null) {
	            throw new NotFoundException(30003);
	        }
	        return spu;
	    }
	
	    @GetMapping("/id/{id}/simplify")
	    public SpuSimplifyVO getSimplifyVO(@PathVariable @Positive Long id) {
	        Spu spu = spuService.getSpu(id);
	        SpuSimplifyVO vo = new SpuSimplifyVO();
	        BeanUtils.copyProperties(spu, vo);
	        return vo;
	    }
	
	    @GetMapping("/latest")
	    public PagingDozer<Spu, SpuSimplifyVO> getLatestSpuList(@RequestParam(defaultValue = "0") Integer start,
	                                                @RequestParam(defaultValue = "10") Integer count) {
	        PageCounter pageCounter = CommonUtil.convertToPageParameter(start, count);
	        Page<Spu> page = spuService.getLatestPagingSpu(pageCounter.getPage(), pageCounter.getCount());
	        return new PagingDozer<>(page, SpuSimplifyVO.class);
	
	    }
	}


### 无限级分类表设计,路径表达法  ###
传统的分类中,对于分类节点,我们通常都是采取parent_id字段来表示,但是这种方式对于多级分类来说,回溯查询效率比较低下。我们可以采取设计一个字段path,在path字段中,保存从根节点开始到当前节点的id路径。

### 通过分类id查找对应的Spu  ###
SpuRepository类:

	package com.zhqx.missyou.repository;
	
	import com.zhqx.missyou.model.Spu;
	import org.springframework.data.domain.Page;
	import org.springframework.data.domain.Pageable;
	import org.springframework.data.jpa.repository.JpaRepository;
	import org.springframework.stereotype.Repository;
	
	@Repository
	public interface SpuRepository extends JpaRepository<Spu, Long> {
	    Spu findOneById(Long id);
	
	    Page<Spu> findByCategoryIdOOrderByCreateTimeDesc(Long cid, Pageable pageable);
	
	    Page<Spu> findByRootCategoryIdOrderBOrderByCreateTimeDesc(Long cid, Pageable pageable);
	}

SpuServiceImpl类:	

	package com.zhqx.missyou.service.impl;
	
	import com.zhqx.missyou.model.Spu;
	import com.zhqx.missyou.repository.SpuRepository;
	import com.zhqx.missyou.service.SpuService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.domain.Page;
	import org.springframework.data.domain.PageRequest;
	import org.springframework.data.domain.Pageable;
	import org.springframework.data.domain.Sort;
	import org.springframework.stereotype.Service;
	
	@Service
	public class SpuServiceImpl implements SpuService {
	
	    @Autowired
	    private SpuRepository spuRepository;
	
	    @Override
	    public Spu getSpu(Long id) {
	        return spuRepository.findOneById(id);
	    }
	
	    @Override
	    public Page<Spu> getLatestPagingSpu(Integer pageNum, Integer size) {
	        //这里的排序字段对应的应该是实体中的属性,而不是数据库中的字段
	        Pageable pageable = PageRequest.of(pageNum, size, Sort.by("createTime").descending());
	        return spuRepository.findAll(pageable);
	    }
	
	    @Override
	    public Page<Spu> getByCategory(Long cid, Boolean isRoot, Integer pageNum, Integer size) {
	        Pageable pageable = PageRequest.of(pageNum, size);
	        if (isRoot) {
	            return spuRepository.findByRootCategoryIdOrderBOrderByCreateTimeDesc(cid, pageable);
	        }
	        return spuRepository.findByCategoryIdOOrderByCreateTimeDesc(cid, pageable);
	    }
	}

SpuController类:

	package com.zhqx.missyou.api.v1;
	
	import com.zhqx.missyou.bo.PageCounter;
	import com.zhqx.missyou.exception.http.NotFoundException;
	import com.zhqx.missyou.model.Spu;
	import com.zhqx.missyou.service.SpuService;
	import com.zhqx.missyou.util.CommonUtil;
	import com.zhqx.missyou.vo.PagingDozer;
	import com.zhqx.missyou.vo.SpuSimplifyVO;
	import org.springframework.beans.BeanUtils;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.domain.Page;
	import org.springframework.validation.annotation.Validated;
	import org.springframework.web.bind.annotation.*;
	
	import javax.validation.constraints.Positive;
	
	@RestController
	@RequestMapping("/spu")
	@Validated
	public class SpuController {
	
	    @Autowired
	    private SpuService spuService;
	
	    @GetMapping("/id/{id}/detail")//@Positive整数
	    public Spu getDetail(@PathVariable @Positive Long id) {
	        Spu spu = spuService.getSpu(id);
	        if(spu == null) {
	            throw new NotFoundException(30003);
	        }
	        return spu;
	    }
	
	    @GetMapping("/id/{id}/simplify")
	    public SpuSimplifyVO getSimplifyVO(@PathVariable @Positive Long id) {
	        Spu spu = spuService.getSpu(id);
	        SpuSimplifyVO vo = new SpuSimplifyVO();
	        BeanUtils.copyProperties(spu, vo);
	        return vo;
	    }
	
	    @GetMapping("/latest")
	    public PagingDozer<Spu, SpuSimplifyVO> getLatestSpuList(@RequestParam(defaultValue = "0") Integer start,
	                                                @RequestParam(defaultValue = "10") Integer count) {
	        PageCounter pageCounter = CommonUtil.convertToPageParameter(start, count);
	        Page<Spu> page = spuService.getLatestPagingSpu(pageCounter.getPage(), pageCounter.getCount());
	        return new PagingDozer<>(page, SpuSimplifyVO.class);
	
	    }
	
	    @GetMapping("/by/category/{id}")
	    public PagingDozer<Spu, SpuSimplifyVO> getByCategoryId(@PathVariable @Positive Long id,
	                                                           @RequestParam(name = "is_root", defaultValue = "false") Boolean isRoot,
	                                                           @RequestParam(defaultValue = "0") Integer start,
	                                                           @RequestParam(defaultValue = "10") Integer count) {
	        PageCounter pageCounter = CommonUtil.convertToPageParameter(start, count);
	        Page<Spu> page = spuService.getByCategory(id, isRoot, pageCounter.getPage(), pageCounter.getCount());
	        return new PagingDozer<>(page, SpuSimplifyVO.class);
	    }
	}

### @Validated验证信息的配置  ###
在resources目录下新建配置文件名为ValidationMessages.properties(固定名称)

	id.positive = id必须是正整数

在SpuController中配置验证错误信息

	package com.zhqx.missyou.api.v1;
	
	import com.zhqx.missyou.bo.PageCounter;
	import com.zhqx.missyou.exception.http.NotFoundException;
	import com.zhqx.missyou.model.Spu;
	import com.zhqx.missyou.service.SpuService;
	import com.zhqx.missyou.util.CommonUtil;
	import com.zhqx.missyou.vo.PagingDozer;
	import com.zhqx.missyou.vo.SpuSimplifyVO;
	import org.springframework.beans.BeanUtils;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.domain.Page;
	import org.springframework.validation.annotation.Validated;
	import org.springframework.web.bind.annotation.*;
	
	import javax.validation.constraints.Positive;
	
	@RestController
	@RequestMapping("/spu")
	@Validated
	public class SpuController {
	
	    @Autowired
	    private SpuService spuService;
	
	    @GetMapping("/id/{id}/detail")//@Positive整数
	    public Spu getDetail(@PathVariable @Positive(message = "{id.positive}") Long id) {
	        Spu spu = spuService.getSpu(id);
	        if(spu == null) {
	            throw new NotFoundException(30003);
	        }
	        return spu;
	    }
	
	    @GetMapping("/id/{id}/simplify")
	    public SpuSimplifyVO getSimplifyVO(@PathVariable @Positive(message = "{id.positive}") Long id) {
	        Spu spu = spuService.getSpu(id);
	        SpuSimplifyVO vo = new SpuSimplifyVO();
	        BeanUtils.copyProperties(spu, vo);
	        return vo;
	    }
	
	    @GetMapping("/latest")
	    public PagingDozer<Spu, SpuSimplifyVO> getLatestSpuList(@RequestParam(defaultValue = "0") Integer start,
	                                                @RequestParam(defaultValue = "10") Integer count) {
	        PageCounter pageCounter = CommonUtil.convertToPageParameter(start, count);
	        Page<Spu> page = spuService.getLatestPagingSpu(pageCounter.getPage(), pageCounter.getCount());
	        return new PagingDozer<>(page, SpuSimplifyVO.class);
	
	    }
	
	    @GetMapping("/by/category/{id}")
	    public PagingDozer<Spu, SpuSimplifyVO> getByCategoryId(@PathVariable @Positive(message = "{id.positive}") Long id,
	                                                           @RequestParam(name = "is_root", defaultValue = "false") Boolean isRoot,
	                                                           @RequestParam(defaultValue = "0") Integer start,
	                                                           @RequestParam(defaultValue = "10") Integer count) {
	        PageCounter pageCounter = CommonUtil.convertToPageParameter(start, count);
	        Page<Spu> page = spuService.getByCategory(id, isRoot, pageCounter.getPage(), pageCounter.getCount());
	        return new PagingDozer<>(page, SpuSimplifyVO.class);
	    }
	}
