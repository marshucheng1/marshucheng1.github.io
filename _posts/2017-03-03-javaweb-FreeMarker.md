---
layout: post
title: FreeMarker相关使用内容
date: 2017-03-03 13:31:22
categories: JavaWeb
tags: FreeMarker
author: MarsHu
---

* content
{:toc}

# SpringBoot配置FreeMarker模板 #

	package com.hyhb.configs;
	
	import cn.org.rapid_framework.freemarker.directive.BlockDirective;
	import cn.org.rapid_framework.freemarker.directive.ExtendsDirective;
	import cn.org.rapid_framework.freemarker.directive.OverrideDirective;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer;
	
	import javax.annotation.PostConstruct;
	
	@Configuration
	public class FreemarkerConfig {
	
	    @Autowired
	    freemarker.template.Configuration configuration;
	    @Autowired
	    private FreeMarkerConfigurer freeMarkerConfigurer;
	
	    @PostConstruct
	    public void setSharedVariable() {
	        //freemarker模板继承
	        configuration.setSharedVariable("block", new BlockDirective());
	        configuration.setSharedVariable("override", new OverrideDirective());
	        configuration.setSharedVariable("extends", new ExtendsDirective());
	    }
	}



# FreeMarker Map的key值必须是String类型 #

官方明确说明了建议使用String类型做为key值

	map集合遍历：key代表map的key。
	<#list customerType?keys as key >
	<option value="${key}">${customerType[key]}</option>
	</#list>

# freemark 日期转换 #

	${item.lastLoginDate?string("yyyy-MM-dd")}

# freemark 三元表达式 #

	${(update_name_error??) ? string("has-error","")}

# freemarker将数字转换为字符串 #

	${123?c} 结果为123 

# freemark判断是否包含字符串 #

	${(form.getMessageFrom()!'') ?contains(key) ? string("checked", "") }

# freemark自动将多位数数字处理成字符串,以","隔开解决 #

	在模板中直接加.toString()转化数字为字符串，如 ${num.toString()}  

	* 使用?c控制，如 ${num?c}
	* 在freemarker配置文件freemarker.properties(在class目录下即可)加number_format=# 
	* 在模板中直接加<#setting number_format="#">;
	* 通过freemarker.template.Configuration的config.setNumberFormat("#")来设定freemarker对数值的格式化;

# SpringBoot导入支持jsp标签的jar包 # 

	<dependency>
	   <groupId>commons-lang</groupId>
	   <artifactId>commons-lang</artifactId>
	   <version>2.6</version>
	</dependency>
	<dependency>
	   <groupId>javax.servlet.jsp</groupId>
	   <artifactId>jsp-api</artifactId>
	   <version>2.1</version>
	</dependency>	

# freemark使用模板继承,并使用自定义分页标签 #

	package com.hyhb.configs;
	
	import cn.org.rapid_framework.freemarker.directive.BlockDirective;
	import cn.org.rapid_framework.freemarker.directive.ExtendsDirective;
	import cn.org.rapid_framework.freemarker.directive.OverrideDirective;
	import freemarker.ext.jsp.TaglibFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer;
	
	import javax.annotation.PostConstruct;
	import java.util.ArrayList;
	import java.util.List;
	
	@Configuration
	public class FreemarkerConfig {
	
	    @Autowired
	    freemarker.template.Configuration configuration;
	    @Autowired
	    private FreeMarkerConfigurer freeMarkerConfigurer;
	
	    @PostConstruct
	    public void setSharedVariable() {
	        //freemarker模板继承
	        configuration.setSharedVariable("block", new BlockDirective());
	        configuration.setSharedVariable("override", new OverrideDirective());
	        configuration.setSharedVariable("extends", new ExtendsDirective());
	        //freemarker自定义标签
	        List<String> tlds = new ArrayList<>();
	        //标签存放的位置
	        tlds.add("/static/tags/page.tld");
	        TaglibFactory taglibFactory = freeMarkerConfigurer.getTaglibFactory();
	        taglibFactory.setClasspathTlds(tlds);
	        if (taglibFactory.getObjectWrapper() == null) {
	            taglibFactory.setObjectWrapper(freeMarkerConfigurer.getConfiguration().getObjectWrapper());
	        }
	    }
	}

# 分页标签page.tld #

	<?xml version="1.0" encoding="UTF-8"?>
	<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
			version="2.0">
		<description>分页标签</description>
		<tlib-version>1.0</tlib-version>
		<short-name>page</short-name>
		<uri>/page</uri>
		<tag>
			<description>
				用于页面分页的输出
			</description>
			<name>page</name>
			<tag-class>com.hyhb.utils.PageTag</tag-class>
			<body-content>empty</body-content>
			<attribute>
				<description>分页需要的对象</description>
				<name>pagination</name>
				<required>true</required>
				<rtexprvalue>true</rtexprvalue>
			</attribute>
			<attribute>
				<description>是否前端页面</description>
				<name>isWeb</name>
				<required>false</required>
				<rtexprvalue>true</rtexprvalue>
			</attribute>
		</tag>
	</taglib>

# 分页类-PageTag #

	package com.hyhb.utils;
	
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.jsp.JspException;
	import javax.servlet.jsp.JspWriter;
	import javax.servlet.jsp.PageContext;
	import javax.servlet.jsp.tagext.SimpleTagSupport;
	import java.io.IOException;
	import java.util.HashMap;
	import java.util.Map;
	
	public class PageTag extends SimpleTagSupport {
	    private Pagination pagination;
	    private boolean isWeb;
	
	    private String getPageUrl(Integer page) {
	
	        HttpServletRequest request = (HttpServletRequest) ((PageContext) this.getJspContext()).getRequest();
	        String url = this.pagination.getUrl();
	        String queryString = request.getQueryString();
	        url = request.getContextPath() + url;
	        Map<String, String> params = new HashMap<String, String>();
	        params.put("page", page.toString());
	        String refactorString = this.pagination.refactorParams(queryString, params);
	        if (refactorString.length() > 0) {
	            url = url + "?" + refactorString;
	        }
	        return url;
	    }
	
	    private void getDashboardPages(StringBuffer sb) {
	        sb.append("<div class='container-fluid' style='position: relative;height: 40px;width: 100%;text-align: center;'>");
	        sb.append(String.format("<div style='position: absolute;left: 0;top: 0;width: auto;'>共有 %d 条记录</div>", this.pagination.getTotalCount()));
	        sb.append("<ul class='pagination' style='margin: auto;width: auto;'>");
	        if (this.pagination.hasPrev()) {
	            sb.append(String.format("<li><a href='%s' aria-label='Previous'> <span aria-hidden='true'>&laquo;</span></a></li>", this.getPageUrl(this.pagination.getPage() - 1)));
	        } else {
	            sb.append("<li class='disabled'><a href='javascript:;' aria-label='Previous'><span aria-hidden='true'>&laquo;</span></a></li>");
	        }
	        for (Integer page : this.pagination.iteratorPages()) {
	            if (page > 0) {
	                if (page != this.pagination.getPage()) {
	                    sb.append(String.format("<li><a href='%s'>%d</a></li>", this.getPageUrl(page), page));
	                } else {
	                    sb.append(String.format("<li class='active'><a href='javascript:;'>%d</a></li>", page));
	                }
	            } else {
	                sb.append("<li class='disabled'><a href='javascript:;'>...</a></li>");
	            }
	        }
	        if (this.pagination.hasNext()) {
	            sb.append(String.format("<li><a href='%s' aria-label='Previous'> <span aria-hidden='true'>&raquo;</span></a></li>", this.getPageUrl(this.pagination.getPage() + 1)));
	        } else {
	            sb.append("<li class='disabled'><a href='javascript:;' aria-label='Previous'><span aria-hidden='true'>&raquo;</span></a></li>");
	        }
	        sb.append("</ul>");
	        sb.append("</div>");
	    }
	
	    private void getWebPages(StringBuffer sb) {
	        sb.append("<div class='Pages'>");
	        sb.append("<span class='p_page'>");
	        if (this.pagination.getPages() > 0) {
	            sb.append(String.format("<a href='%s' class='a_prev'>首页</a>", this.getPageUrl(1)));
	        } else {
	            sb.append("<a href='javascript:void(0);' class='a_prev'>首页</a>");
	        }
	        if (this.pagination.hasPrev()) {
	            sb.append(String.format("<a href='%s' class='a_prev' style='margin:0 2px;'>上一页</a>", this.getPageUrl(this.pagination.getPage() - 1)));
	        } else {
	            sb.append("<a href='javascript:void(0);' class='a_prev' style='margin:0 2px;'>上一页</a>");
	        }
	        sb.append("<em class='num'>");
	        for (Integer page :
	                this.pagination.iteratorWebPages()) {
	            if (page != this.pagination.getPage()) {
	                sb.append(String.format("<a href='%s' class='a_num'>%d</a>", this.getPageUrl(page), page));
	            } else {
	                sb.append(String.format("<a href='javascript:void(0);' class='a_cur'>%d</a>", page));
	            }
	        }
	        sb.append("</em>");
	        if (this.pagination.hasNext()) {
	            sb.append(String.format("<a href='%s' class='a_next'>下一页</a>", this.getPageUrl(this.pagination.getPage() + 1)));
	        } else {
	            sb.append("<a href='javascript:void(0);' class='a_next'>下一页</a>");
	        }
	        sb.append("</span>");
	        sb.append("</div>");
	    }
	
	    @Override
	    public void doTag() throws JspException {
	        JspWriter out = this.getJspContext().getOut();//指定输入流，用于页面输出分页信息、
	
	        StringBuffer sb = new StringBuffer();//构建StringBuffer对象，用户拼接分页标签
	        if (this.isWeb) {
	            getWebPages(sb);
	        } else {
	            getDashboardPages(sb);
	        }
	        try {
	            out.print(sb.toString());
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	    }
	
	    public Pagination getPagination() {
	        return pagination;
	    }
	
	    public void setPagination(Pagination pagination) {
	        this.pagination = pagination;
	    }
	
	    public boolean getIsWeb() {
	        return isWeb;
	    }
	
	    public void setIsWeb(boolean isWeb) {
	        this.isWeb = isWeb;
	    }
	}

# 分页实体类-Pagination #

	package com.hyhb.utils;
	
	import java.util.ArrayList;
	import java.util.HashMap;
	import java.util.List;
	import java.util.Map;
	
	public class Pagination {
	
	    private Integer page;
	    private Integer totalCount;
	    private Integer pageSize;
	    private String url;
	
	    /**
	     * 分页公用类
	     *
	     * @param page       Integer 页数
	     * @param pageSize   Integer 页面大小
	     * @param totalCount Integer 总数据大小
	     * @param url        Integer 分页url
	     */
	    public Pagination(Integer page, Integer pageSize, Integer totalCount, String url) {
	        this.page = page;
	        this.pageSize = pageSize;
	        this.totalCount = totalCount;
	        this.url = url;
	    }
	
	    public Integer getPages() {
	        return (this.totalCount + this.pageSize - 1) / this.pageSize;
	    }
	
	    public Boolean hasPrev() {
	        return this.page > 1;
	    }
	
	    public Boolean hasNext() {
	        return this.page < this.getPages();
	    }
	
	    public List<Integer> iteratorPages() {
	        Integer leftEdge = 2;
	        Integer leftCurrent = 2;
	        Integer rightCurrent = 5;
	        Integer rightEdge = 2;
	        List<Integer> list = new ArrayList<Integer>();
	        for (Integer num = 1; num <= this.getPages(); num++) {
	            if (num <= leftEdge || (num > this.page - leftCurrent - 1 && num < this.page + rightCurrent) || num > this.getPages() - rightEdge) {
	                list.add(num);
	            } else {
	                if (list.size() > 0 && list.get(list.size() - 1) > 0) {
	                    list.add(-1);
	                }
	            }
	        }
	        return list;
	    }
	
	    public List<Integer> iteratorWebPages() {
	        Integer leftEdge = 4;
	        Integer rightEdge = 4;
	        List<Integer> pages = new ArrayList<>();
	        int leftCurrent = 5 - this.page > 0 ? 5 - this.page : 0;
	        rightEdge = rightEdge + leftCurrent;
	        int rightCurrent = 5 + page > this.getPages() ? 5 + page - this.getPages() : 0;
	        leftEdge = leftEdge + rightCurrent;
	        for (int num = 1; num <= this.getPages(); num++) {
	            if ((num >= page - leftEdge && num <= page + rightEdge)) {
	                pages.add(num);
	            }
	        }
	        return pages;
	    }
	
	    public String refactorParams(String queryString, Map<String, String> params) {
	        Map<String, String> allParams = new HashMap<String, String>();
	        if (null != queryString && queryString.length() > 0) {
	            String[] paramList = queryString.split("&");
	            for (int i = 0; i < paramList.length; i++) {
	                String[] key_vale = paramList[i].split("=");
	                if (key_vale.length > 1) {
	                    allParams.put(key_vale[0], key_vale[1]);
	                } else {
	                    allParams.put(key_vale[0], "");
	                }
	            }
	        }
	        if (null != params) {
	            allParams.putAll(params);
	        }
	        String refactorString = "";
	        for (String key : allParams.keySet()) {
	            if (refactorString.length() > 0) {
	                refactorString += "&";
	            }
	            refactorString = refactorString + key + "=" + allParams.get(key);
	        }
	        return refactorString;
	    }
	
	    public Integer getPage() {
	        return page;
	    }
	
	    public void setPage(Integer page) {
	        this.page = page;
	    }
	
	    public Integer getTotalCount() {
	        return totalCount;
	    }
	
	    public void setTotalCount(Integer totalCount) {
	        this.totalCount = totalCount;
	    }
	
	    public Integer getPageSize() {
	        return pageSize;
	    }
	
	    public void setPageSize(Integer pageSize) {
	        this.pageSize = pageSize;
	    }
	
	    public String getUrl() {
	        return url;
	    }
	
	    public void setUrl(String url) {
	        this.url = url;
	    }
	
	    public static void main(String[] args) {
	        Pagination pagination = new Pagination(8, 20, 4000, "");
	        Map<String, String> params = new HashMap<String, String>();
	        params.put("param1", "param1");
	        params.put("param2", "param2");
	        String refactorString = pagination.refactorParams("param1=1&param3=", params);
	        System.out.println(refactorString);
	    }
	}


# freemark页面中使用分页标签 #

页面头部导入标签

	<#assign page = JspTaglibs["/page"] />

在需要分页的地方使用-后端

	<@page.page pagination=pagination/>

如果是前端

	<@page.page pagination=pagination isWeb=true/>