---
layout: post
title: springboot邮件发送
date: 2017-03-02 13:31:22
categories: JavaWeb
tags: springboot
author: MarsHu
---

* content
{:toc}

# springboot版本2.1.3,maven-web项目,pom文件 #
	<!-- 邮件发送jar包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>
	<!-- 邮件模板thymeleaf -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
教程学习于慕课网,课程链接:[https://www.imooc.com/learn/1036](https://www.imooc.com/learn/1036 "Spring Boot 发送邮件")





# 发送人邮件配置,本人使用的是网易163邮箱,可以替换成其他的 #
	spring:
	  mail:
	    host: smtp.163.com
	    username: 账号
	    password: 密码
	    default-encoding: utf-8

# 发送邮件Service #
	package ah.zhuoshan.service;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.core.io.FileSystemResource;
	import org.springframework.mail.SimpleMailMessage;
	import org.springframework.mail.javamail.JavaMailSender;
	import org.springframework.mail.javamail.MimeMessageHelper;
	import org.springframework.stereotype.Service;
	
	import javax.mail.MessagingException;
	import javax.mail.internet.MimeMessage;
	import java.io.File;
	
	@Service
	public class MailService {
	
	    /*发送人*/
	    @Value("${spring.mail.username}")
	    private String from;
	
	    @Autowired
	    private JavaMailSender mailSender;
	
	    /**
	     * 发送简单文本邮件
	     * @param to 收件人
	     * @param subject 邮件主题
	     * @param text 邮件内容
	     */
	    public void sendSimpleEmailMessage(String to, String subject, String text) {
	        SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
	
	        simpleMailMessage.setTo(to);
	        simpleMailMessage.setSubject(subject);
	        simpleMailMessage.setText(text);
	        simpleMailMessage.setFrom(from);
	        /*发送邮件*/
	        mailSender.send(simpleMailMessage);
	    }
	
	    /**
	     * 发送简单html邮件
	     * @param to 收件人
	     * @param subject 邮件主题
	     * @param text 邮件内容
	     * @throws MessagingException
	     */
	    public void sendHtmlMail(String to, String subject, String text) throws MessagingException {
	        MimeMessage mimeMessage = mailSender.createMimeMessage();
	
	        MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, true);
	
	        mimeMessageHelper.setTo(to);
	        mimeMessageHelper.setSubject(subject);
	        mimeMessageHelper.setText(text, true);
	        mimeMessageHelper.setFrom(from);
	
	        mailSender.send(mimeMessage);
	    }
	
	    /**
	     * 发送带附件邮件
	     * @param to 收件人
	     * @param subject 邮件主题
	     * @param text 邮件内容
	     * @param filePath 附件地址
	     * @throws MessagingException
	     */
	    public void sendAttachmentMail(String to, String subject, String text, String filePath) throws MessagingException {
	        MimeMessage mimeMessage = mailSender.createMimeMessage();
	
	        MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, true);
	        mimeMessageHelper.setTo(to);
	        mimeMessageHelper.setSubject(subject);
	        mimeMessageHelper.setText(text, true);
	        mimeMessageHelper.setFrom(from);
	
	        FileSystemResource fileSystemResource = new FileSystemResource(new File(filePath));
	
	        String fileName = fileSystemResource.getFilename();
	
	        mimeMessageHelper.addAttachment(fileName, fileSystemResource);
	        /*第二个附件*/
	        mimeMessageHelper.addAttachment(fileName + "_test", fileSystemResource);
	        /*实际开发中,如果需要发送多个附件,可以String filePath 参数改为 String[] filePaths 然后遍历即可*/
	        mailSender.send(mimeMessage);
	    }
	
	    /**
	     * 发送图片内容邮件
	     * @param to 收件人
	     * @param subject 邮件主题
	     * @param text 邮件内容
	     * @param imagePath 图片地址
	     * @param imageId 区分不同图片的id
	     */
	    public void sendImageMail(String to, String subject, String text, String imagePath, String imageId) throws MessagingException {
	        MimeMessage mimeMessage = mailSender.createMimeMessage();
	
	        MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, true);
	        mimeMessageHelper.setTo(to);
	        mimeMessageHelper.setSubject(subject);
	        mimeMessageHelper.setText(text, true);
	        mimeMessageHelper.setFrom(from);
	
	        FileSystemResource fileSystemResource = new FileSystemResource(new File(imagePath));
	
	        mimeMessageHelper.addInline(imageId, fileSystemResource);
	        /*第二张图片*/
	        mimeMessageHelper.addInline(imageId, fileSystemResource);
	
	        mailSender.send(mimeMessage);
	    }
	
	}

# 邮件发送测试方法,这里将目标邮箱xxx替换成自己的目标邮箱即可 #

这里,顺带提一下,`markdown`因为支持`html`标签,所以当我们需要插入`html`作为代码时,

会出现一些极度不友好的错误提示,意思是说`markdown`不知道到底是要显示`html`还是作为代码显示。

在本例中,稍作了改变,每个`String`变量中的`<html>`标签,左边部分使用`&lt;`代替,这样就不报错了。

复制代码时,**改过来即可**。感觉很弱智啊!

	package ah.zhuoshan.service;
	
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.context.SpringBootTest;
	import org.springframework.test.context.junit4.SpringRunner;
	import org.thymeleaf.TemplateEngine;
	import org.thymeleaf.context.Context;
	
	import javax.mail.MessagingException;
	
	import static org.junit.Assert.*;
	
	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class MailServiceTest {
	
	    @Autowired
	    private MailService mailService;
	
	    @Autowired
	    private TemplateEngine templateEngine;
	
	    @Test
	    public void sendSimpleEmailMessage() {
	        mailService.sendSimpleEmailMessage("目标邮箱xxx", "小马哥,这不是垃圾邮件", "springboot-mail:简单文本邮件发送");
	    }
	
	    @Test
	    public void sendHtmlMail() throws MessagingException {
	        String html_text = "<!DOCTYPE html>\n" +
                "&lt;html lang=\"en\">\n" +
                "<head>\n" +
                "    <meta charset=\"UTF-8\">\n" +
                "    <title>html邮件</title>\n" +
                "</head>\n" +
                "<body>\n" +
                "<h1>springboot-mail:简单html邮件发送</h1>\n" +
                "</body>\n" +
                "</html>";
	
	        mailService.sendHtmlMail("目标邮箱xxx", "小马哥,强无敌", html_text);
	    }
	
	    @Test
	    public void sendAttachmentMail() throws MessagingException {
	        String filePath = "C:\\Users\\hucheng\\Desktop\\applicationContext.rar";
	        mailService.sendAttachmentMail("目标邮箱xxx", "小马哥,再这样我要翻脸了", "spring配置文件附件", filePath);
	    }
	
	    /**
	     * 发送多张不同图片
	     * 1.创建多个img标签--[这里请仔细拼接cid......]
	     * 2.创建多个imagePath、和imageId
	     * 3.在方法中使用多个addInline(imageId, fileSystemResource);方法
	     * 4.这里以相同图片作为示例
	     * @throws MessagingException
	     */
	    @Test
	    public void sendImageMail() throws MessagingException {
	        String imagePath = "C:\\Users\\hucheng\\Desktop\\test.png";
	        String imageId = "test001";
	        String html_text = "<!DOCTYPE html>\n" +
                "&lt;html lang=\"en\">\n" +
                "<head>\n" +
                "    <meta charset=\"UTF-8\">\n" +
                "    <title>html邮件</title>\n" +
                "</head>\n" +
                "<body>\n" +
                "<img src=\"cid:" + imageId + "\"/>\n" +
                "<br/>" +
                "<img src=\"cid:" + imageId + "\"/>\n" +
                "</body>\n" +
                "</html>";
	        mailService.sendImageMail("目标邮箱xxx", "小马哥,别说话,吻我", html_text, imagePath, imageId);
	    }
	
	    @Test
	    public void templateMailTest() throws MessagingException {
	        Context context = new Context();
	        context.setVariable("id", "006");
	
	        String emailContent = templateEngine.process("emailTemplate", context);
	
	        mailService.sendHtmlMail("目标邮箱xxx", "南山法院来战", emailContent);
	    }
	}


# thymeleaf邮件发送模板,地址是视频作者博客地址,有兴趣的同学可以研究一些gayhub博客 #

第二个很弱智的地方...下面的代码如果你是直接从html代码复制过来,那么对不起,肯定报错。
但是如果你一行一行粘贴过来,弱智的通过了编写~~~

	<!DOCTYPE html>
	<html lang="en" xmlns:th="http://www.thymeleaf.org">
	<head>
	<meta charset="UTF-8">
	<title>邮件模板</title>
	</head>
	<body>
	<p>你好,感谢您的注册,这是一封验证邮件,请点击下面的连接完成注册,感谢您的支持!</p>
	<a href="#" th:href="@{http://www.ityouknow.com/register/{id}(id=${id})}">激活账号</a>
	</body>
	</html>