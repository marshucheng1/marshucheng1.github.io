---
layout: post
title: Springboot-单体JSON对象的处理
date: 2021-02-20 12:10:22
categories: springboot
tags: springboot
author: MarsHu
---

* content
{:toc}

### json字符格式字段数据返回内容  ###
观察数据库表sku,字段specs的格式为json类型,值格式为如下内容:

	[{"key": "颜色", "value": "青蓝色", "key_id": 1, "value_id": 1}, {"key": "尺寸", "value": "7英寸", "key_id": 2, "value_id": 5}]

我们可以观察得出,其值为json格式数组,并且每个元素应该代表一个json对象。我们在Sku实体中是以private String specs;形式去接收的。

当我们访问请求localhost:8080/v1/spu/id/28/detail,因为我们在Spu中设置了关联关系,所以得到如下有关skuList结果片段:

	        {
	            "id": 43,
	            "price": 999.00,
	            "discount_price": null,
	            "online": true,
	            "img": "http://xxx.xxx.xxx/7c462cf2-3874-4ecc-85f9-6b4a0fde4623.png",
	            "title": "Ins复古金色落地灯（落地灯）",
	            "spu_id": 28,
	            "specs": "[{\"key\": \"颜色\", \"value\": \"金色\", \"key_id\": 1, \"value_id\": 27}, {\"key\": \"台灯高低\", \"value\": \"落地灯\", \"key_id\": 8, \"value_id\": 38}]",
	            "code": "28$1-27#8-38",
	            "stock": 19,
	            "category_id": 23,
	            "root_category_id": 4
	        }







在上面的返回结果中,我们可以看到,我们当前的Sku类的specs字段是以如下形式返回的。该返回结果并不利于实际观察

	"specs": "[{\"key\": \"颜色\", \"value\": \"金色\", \"key_id\": 1, \"value_id\": 27}, {\"key\": \"台灯高低\", \"value\": \"落地灯\", \"key_id\": 8, \"value_id\": 38}]",

我们我们实际上是希望该字段能以对象的形式返回结果,Spec类如下,Spec对象对应json数组中每个元素表示的json对象。

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	@Getter
	@Setter
	public class Spec {
	    private Long keyId;
	    private String key;
	    private Long valueId;
	    private String value;
	}

为了便于更好的观察单一json字符串与Spec对象的封装转化,我们可以在Sku表中新增一个字段test,字段数据类型为json,并且将第一条数据specs字段的其中一个json对象填入字段内容。

	{"key": "颜色", "value": "青蓝色", "key_id": 1, "value_id": 1}

同时,我们修改Sku类,以Map<String, Object> test来接收test字段内容:

	package com.zhqx.missyou.model;
	
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.Entity;
	import javax.persistence.Id;
	import java.math.BigDecimal;
	import java.util.Map;
	
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
	    private Map<String, Object> test;
	    private String code;
	    private Long stock;
	    private Long categoryId;
	    private Long rootCategoryId;
	
	}

修改号后,我们会发现代码出错,这是因为我们还没有进行自定义处理,JPA没有办法自动将json数据封装到对象。

### 单体JSON对象的映射  ###
默认的JPA封装没有办法处理json对象到Map<String, Object>的处理,所以我们需要自定义处理方法MapAndJson.

	package com.zhqx.missyou.util;
	
	import com.fasterxml.jackson.core.JsonProcessingException;
	import com.fasterxml.jackson.databind.ObjectMapper;
	import com.zhqx.missyou.exception.http.ServerErrorException;
	import org.springframework.beans.factory.annotation.Autowired;
	
	import javax.persistence.AttributeConverter;
	import javax.persistence.Converter;
	import java.util.HashMap;
	import java.util.Map;
	
	@Converter
	public class MapAndJson implements AttributeConverter<Map<String, Object>, String> {
	
	    @Autowired
	    private ObjectMapper mapper;
	
	    @Override
	    public String convertToDatabaseColumn(Map<String, Object> stringObjectMap) {
	        try {
	            return mapper.writeValueAsString(stringObjectMap);
	        } catch (JsonProcessingException e) {
	            e.printStackTrace();
	            throw new ServerErrorException(9999);
	        }
	    }
	
	    @Override
	    public Map<String, Object> convertToEntityAttribute(String s) {
	        try {
	            return mapper.readValue(s, HashMap.class);
	        } catch (JsonProcessingException e) {
	            e.printStackTrace();
	            throw new ServerErrorException(9999);
	        }
	    }
	}

ServerErrorException类:

	package com.zhqx.missyou.exception.http;
	
	public class ServerErrorException extends HttpException {
	    public ServerErrorException(int code) {
	        this.code = code;
	        this.httpStatusCode = 500;
	    }
	}

exception-code.properties配置类中配置ServerErrorException异常对应的信息

	zhqx.codes[9999] = 服务器未知异常O(∩_∩)O哈哈~
	
	zhqx.codes[10000]= 通用异常
	zhqx.codes[10001]= 通用参数异常
	
	zhqx.codes[30003]= 商品信息不存在
	zhqx.codes[30006]= Spu类的资源不存在

在Sku类中指定转换方法:

	package com.zhqx.missyou.model;
	
	import com.zhqx.missyou.util.MapAndJson;
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.Convert;
	import javax.persistence.Entity;
	import javax.persistence.Id;
	import java.math.BigDecimal;
	import java.util.Map;
	
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
	    @Convert(converter = MapAndJson.class)
	    private Map<String, Object> test;
	    private String code;
	    private Long stock;
	    private Long categoryId;
	    private Long rootCategoryId;
	
	}

由于我们修改的Sku数据的spu对应的id为1,所以访问:localhost:8080/v1/spu/id/1/detail,的到如下结果:

	{
	    "id": 1,
	    "title": "青锋大碗",
	    "subtitle": "大碗主要用来盛宽面，凡凡倾情推荐",
	    "category_id": 28,
	    "root_category_id": 27,
	    "online": true,
	    "price": "12.99",
	    "sketch_spec_id": 1,
	    "default_sku_id": null,
	    "img": "http://xxx.xxx.xxx/32ba82d0-4fbe-4bbb-b833-fdc8d397bb34.png",
	    "discount_price": "11.11",
	    "description": null,
	    "tags": "林白推荐",
	    "is_test": true,
	    "for_theme_img": "http://xxx.xxx.xxx/c38dd758-e07a-46d4-a1be-3a4601f886f9.png",
	    "sku_list"
	}{
	    "code": 9999,
	    "message": "服务器异常",
	    "request": "GET /v1/spu/id/1/detail"
	}

在当前Spu的private List<Sku> skuList;属性使用的是懒加载模式,即程序用到skuList时,才会查询结果。在Springboot2.5.2中,会出现如上结果,如果改为Springboot2.2.2,则只会出现如下结果

	{
	    "code": 9999,
	    "message": "服务器异常",
	    "request": "GET /v1/spu/id/1/detail"
	}

或者对Spu类中采取热加载模式时

	@OneToMany(fetch = FetchType.EAGER)
    @JoinColumn(name = "spuId")
    private List<Sku> skuList;

也只会出现如下结果:

	{
	    "code": 9999,
	    "message": "服务器异常",
	    "request": "GET /v1/spu/id/1/detail"
	}

**备注:在Springboot2.3.2(不包括)以后的版本中,应该做了相关返回数据的改动,导致统一异常处理没能拦截到全部数据。**

这所以会出现这样的错误,是因为数据库中的数据没有补全,我们将Sku表中Spu的id与第一条相同的sku数据的test字段填充上内容。

修改正常后,我们再次访问时,可以看到对应数据格式片段如下:

	"test": {
                "key_id": 1,
                "value_id": 1,
                "value": "青蓝色",
                "key": "颜色"
            },


### 数组类型JSON与List的映射  ###
自定义处理方法ListAndJson.

	package com.zhqx.missyou.util;
	
	import com.fasterxml.jackson.core.JsonProcessingException;
	import com.fasterxml.jackson.databind.ObjectMapper;
	import com.zhqx.missyou.exception.http.ServerErrorException;
	import org.springframework.beans.factory.annotation.Autowired;
	
	import javax.persistence.AttributeConverter;
	import javax.persistence.Converter;
	import java.util.List;
	
	@Converter
	public class ListAndJson implements AttributeConverter<List<Object>, String> {
	    @Autowired
	    private ObjectMapper mapper;
	
	    @Override
	    public String convertToDatabaseColumn(List<Object> objects) {
	        try {
	            return mapper.writeValueAsString(objects);
	        } catch (JsonProcessingException e) {
	            e.printStackTrace();
	            throw new ServerErrorException(9999);
	        }
	    }
	
	    @Override
	    public List<Object> convertToEntityAttribute(String s) {
	        try {
	            if(s == null){
	                return null;
	            }
	            List<Object> t = mapper.readValue(s, List.class);
	            return t;
	        } catch (Exception e) {
	            e.printStackTrace();
	            throw new ServerErrorException(9999);
	        }
	    }
	}

修改Sku类:

	@Convert(converter = ListAndJson.class)
    private List<Object> specs;

再次访问上面的请求地址,可以得到如下的代码片段:

	"specs": [
                {
                    "key": "颜色",
                    "value": "青蓝色",
                    "key_id": 1,
                    "value_id": 1
                },
                {
                    "key": "尺寸",
                    "value": "7英寸",
                    "key_id": 2,
                    "value_id": 5
                }
            ],

### 自定义方法实现JSON到对象之间的转换  ###
自定义通用转化工具类GenericAndJson:

	package com.zhqx.missyou.util;
	
	import com.fasterxml.jackson.core.JsonProcessingException;
	import com.fasterxml.jackson.core.type.TypeReference;
	import com.fasterxml.jackson.databind.ObjectMapper;
	import com.zhqx.missyou.exception.http.ServerErrorException;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Component;
	
	@Component
	public class GenericAndJson {
	    private static ObjectMapper mapper;
	
	    @Autowired
	    public void setMapper(ObjectMapper mapper) {
	        GenericAndJson.mapper = mapper;
	    }
	
	    public static <T> String objectToJson(T o) {
	        try {
	            return GenericAndJson.mapper.writeValueAsString(o);
	        } catch (Exception e) {
	            e.printStackTrace();
	            throw new ServerErrorException(9999);
	        }
	    }
	
	    public static <T> T jsonToObject(String s,  TypeReference<T> tr) {
	        if (s == null) {
	            return null;
	        }
	        try {
	            T o = GenericAndJson.mapper.readValue(s, tr);
	            return o;
	        } catch (JsonProcessingException e) {
	            e.printStackTrace();
	            throw new ServerErrorException(9999);
	        }
	    }
	}

修改实体的setter和getter方法:

	package com.zhqx.missyou.model;
	
	import com.fasterxml.jackson.core.type.TypeReference;
	import com.zhqx.missyou.util.GenericAndJson;
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.persistence.Entity;
	import javax.persistence.Id;
	import java.math.BigDecimal;
	import java.util.Collections;
	import java.util.List;
	
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
	
	    public List<Spec> getSpecs() {
	        if (this.specs == null) {
	            return Collections.emptyList();
	        }
	        return GenericAndJson.jsonToObject(this.specs, new TypeReference<List<Spec>>(){});
	    }
	
	    public void setSpecs(List<Spec> specs) {
	        if(specs.isEmpty()){
	            return;
	        }
	        this.specs = GenericAndJson.objectToJson(specs);
	    }
	
	}
