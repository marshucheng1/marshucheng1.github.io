---
layout: post
title: Springboot-定时邮件发送
date: 2017-03-05 15:31:22
categories: JavaWeb
tags: springboot
author: MarsHu
---

* content
{:toc}

### 邮件推送-相关配置设置 ###

这里，我们在用springboot做邮件推送时，大部分情况下，我们都是在配置文件
直接配置的邮件推送设置。如下：

	spring:
	  mail:
	    host: smtp.163.com
	    username: 邮箱名
	    password: 邮箱密码
	    default-encoding: utf-8

我这里使用的是163邮箱。可以根据自己的邮箱进行配置。

	package com.study.constants;
	
	public class Misc {
	    // 推荐消息邮件账户
	    public static final String AUTO_PUSH_MAIL_FROM = "xxxx";
	    // 消息推送邮件标题
	    public static final String AUTO_PUSH_MAIL_SUBJECT = "高大尚的邮件推送主题";
	}

大部分情况下，我们可以不定义邮件账户，因为已经配置了。






### 邮件发送用户 ###
这里只写简单属性。

	package com.study.models;
	
	import java.io.Serializable;
	
	public class User implements Serializable {
	    private Integer id;
	    private String name;
	    private String password;
	    private String email;
	
	    public Integer getId() {
	        return id;
	    }
	
	    public void setId(Integer id) {
	        this.id = id;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    public String getPassword() {
	        return password;
	    }
	
	    public void setPassword(String password) {
	        this.password = password;
	    }
	
	    public String getEmail() {
	        return email;
	    }
	
	    public void setEmail(String email) {
	        this.email = email;
	    }
	}

### 构建邮件实体，方便发送邮件 ###
收件人采用String数组。是为了方便后期如果有群发需求的话。我这里是改造了`setTos`方法，这样处理似乎更加合理。

	package com.study.models;
	
	import com.study.constants.Misc;
	
	import java.util.List;
	
	/**
	 * 邮件发送实体
	 */
	public class MailEntity {
	    private String[] tos;
	    private String subject;
	    private String text;
	    private String from;
	
	    public MailEntity() {
	    }
	
	    /**
	     * 构建单个目标用户的邮件发送模型
	     * @param user 目标用户
	     * @param mailText 邮件内容
	     */
	    public MailEntity(User user, String mailText) {
	        this.from = Misc.AUTO_PUSH_MAIL_FROM;
	        this.tos = new String[]{user.getEmail()};
	        this.subject = Misc.AUTO_PUSH_MAIL_SUBJECT;
	        this.text = mailText;
	    }
	
	    public String[] getTos() {
	        return tos;
	    }
	
	    public void setTos(List<String> tos) {
	        if (tos != null && tos.size() > 0) {
	            String[] toArray = new String[tos.size()];
	            this.tos = tos.toArray(toArray);
	        }
	    }
	
	    public String getSubject() {
	        return subject;
	    }
	
	    public void setSubject(String subject) {
	        this.subject = subject;
	    }
	
	    public String getText() {
	        return text;
	    }
	
	    public void setText(String text) {
	        this.text = text;
	    }
	
	    public String getFrom() {
	        return from;
	    }
	
	    public void setFrom(String from) {
	        this.from = from;
	    }
	}

### 构建异步服务service ###
我们可以在项目中构建一个service，专门用来处理一些需要定时处理、或者异步进行的业务。

	package com.study.service;
	
	import com.study.models.MailEntity;
	
	public interface AsyncTaskService {
	    Boolean sendMailTaskDefault(final MailEntity mailEntity) throws Exception;
	}

异步发送邮件service实现代码。代码部分有什么不明白的可以看看基础邮件发送那里。

	package com.study.service.impl;
	
	import com.study.models.MailEntity;
	import com.study.service.AsyncTaskService;
	import org.springframework.core.task.TaskExecutor;
	import org.springframework.mail.javamail.JavaMailSender;
	import org.springframework.mail.javamail.MimeMessageHelper;
	import org.springframework.stereotype.Service;
	
	import javax.activation.CommandMap;
	import javax.activation.MailcapCommandMap;
	import javax.annotation.Resource;
	import javax.mail.internet.MimeMessage;
	
	@Service
	public class AsyncTaskServiceImpl implements AsyncTaskService {
	    @Resource
	    private JavaMailSender mailSender;
	
	    @Override
	    public Boolean sendMailTaskDefault(MailEntity mailEntity) throws Exception {
	        MimeMessage message = mailSender.createMimeMessage();
	        try {
	            MimeMessageHelper helper = new MimeMessageHelper(message, true, "utf-8");
	            helper.setFrom(mailEntity.getFrom());
	            helper.setTo(mailEntity.getTos());
	            helper.setSubject(mailEntity.getSubject());
	            helper.setText(mailEntity.getText(), true);
	            addSendMailTask(message);
	            return true;
	        } catch (Exception ex) {
	            ex.printStackTrace();
	            return false;
	        }
	    }
	
	    private void addSendMailTask(final MimeMessage message) {
	        MailcapCommandMap mc = (MailcapCommandMap) CommandMap.getDefaultCommandMap();
        	mc.addMailcap("text/html;; x-java-content-handler=com.sun.mail.handlers.text_html");
        	mc.addMailcap("text/xml;; x-java-content-handler=com.sun.mail.handlers.text_xml");
        	mc.addMailcap("text/plain;; x-java-content-handler=com.sun.mail.handlers.text_plain");
        	mc.addMailcap("multipart/*;; x-java-content-handler=com.sun.mail.handlers.multipart_mixed");
        	mc.addMailcap("message/rfc822;; x-java-content-handler=com.sun.mail.handlers.message_rfc822");
        	CommandMap.setDefaultCommandMap(mc);
        	mailSender.send(message);
	    }
	}


### 邮件推送模板 ###
在实际的开发过程中，公司发送邮件的话，可能需要正式一点。因此，我们可以根据需要定义一个邮件模板。

这里任然用springboot推荐的模板引擎之一Freemarker。模板名称：`auto_mail_push.ftl`。

	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <title>自动发送邮件</title>
	</head>
	<body>
	<p>
	    ${username},你好!
	</p>
	</body>
	</html>

这里的`${username}`是从后台传递的。

### 邮件定时推送 ###
这里我们需要获取我们之前定义好的邮件发送模板，根据需要放入适合的数据。然后放入到邮件实体中。

这里设置的10秒钟定时发送，`cron`值有固定的语法格式，根据需要设置（百度）。

	package com.study.quartz;
	
	import com.study.constants.Misc;
	import com.study.models.MailEntity;
	import com.study.service.AsyncTaskService;
	import freemarker.template.Template;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.scheduling.annotation.Scheduled;
	import org.springframework.stereotype.Component;
	import org.springframework.ui.freemarker.FreeMarkerTemplateUtils;
	import org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer;
	
	import javax.annotation.Resource;
	import java.util.ArrayList;
	import java.util.HashMap;
	import java.util.List;
	import java.util.Map;
	
	@Component
	public class QuartPushMail {
	    private Logger logger = LoggerFactory.getLogger(QuartPushMail.class);
	    @Resource
	    private FreeMarkerConfigurer freeMarkerConfigurer;
	    @Resource
	    private AsyncTaskService asyncTaskService;
	    @Value("${spring.mail.username}")
	    private String from;
	
		/*定时10秒发送*/
	    @Scheduled(cron="0/10 * *  * * ? ")
	    public void autoPush() {
	        try {
	            System.out.println("执行邮件发送");
	            sendTask();
	        } catch (Exception e) {
	            e.printStackTrace();
	            logger.info("邮件发送出现异常");
	        }
	    }
	
	    private void sendTask() throws Exception {
	        Map<String, Object> model = new HashMap();
	        List<String> tos = new ArrayList<>();
	        MailEntity mail = new MailEntity();
	        mail.setFrom(from);
	        tos.add("xxx@qq.com");
	        model.put("username", "xxx");
	        mail.setTos(tos);
	        mail.setSubject(Misc.AUTO_PUSH_MAIL_SUBJECT);
	        Template template = freeMarkerConfigurer.getConfiguration().getTemplate("auto_mail_push.ftl");
	        String html_text = FreeMarkerTemplateUtils.processTemplateIntoString(template, model);
	
	        mail.setText(html_text);
	        asyncTaskService.sendMailTaskDefault(mail);
	    }
	}


### springboot启动类加上注解，以支持异步处理 ###
`@EnableScheduling`注解。定时任务支持。

	package com.study;
	
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.scheduling.annotation.EnableScheduling;
	
	@SpringBootApplication
	@EnableScheduling
	public class AsyncsendApplication {
	
	    public static void main(String[] args) {
	        SpringApplication.run(AsyncsendApplication.class, args);
	    }

	}

### 后续问题待补充 ###
上面只是简单能实现定时发送。后续有一些问题需要处理。
