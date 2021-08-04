---
layout: post
title: springcloud（三）
date: 2021-04-18 09:11:22
categories: springcloud
tags: springcloud
author: MarsHu
---

* content
{:toc}

### 使用vue cli创建admin项目  ###
官方地址:https://cli.vuejs.org/zh/.要安装vue-cli的话,我们要确保本机已经安装了node.js,并且版本是符合要求的版本。
在windows的cmd命令窗口执行如下命令:

	npm install -g @vue/cli
	# OR
	yarn global add @vue/cli

如果安装缓慢的话,可以使用淘宝镜像:

	npm install -g cnpm --registry=https://registry.npm.taobao.org

安装完之后使用如下命令查看是否正常安装:

	vue --version

如需升级全局的vue-cli,则使用如下命令(执行上面的安装方法也可以升级):

	npm update -g @vue/cli
	# 或者
	yarn global upgrade --latest @vue/cli






打开idea的命令行窗口,使用vue-cli构建admin项目:
![springcloud3-2.png](http://marshucheng1.github.io/assets/springcloud/springcloud3-2.png)
![springcloud3-3.png](http://marshucheng1.github.io/assets/springcloud/springcloud3-3.png)

构建完成之后,使用如下命令启动vue-admin:
![springcloud3-4.png](http://marshucheng1.github.io/assets/springcloud/springcloud3-4.png)

修改默认启动端口号,当前的vue-cli版本为4.5.13,修改路径为`node_modules/@vue/cli-service/lib/commands/serve.js`

	const defaults = {
	  host: '0.0.0.0',
	  port: 8081,
	  https: false
	}

### 集成bootstrap后台管理模板ace  ###
ace模板下载地址:`https://github.com/bopoda/ace`.下载好文件后,将整个ace文件夹放到public文件夹下.

修改index.html,将Ace模板中的login.html页面的相关js、css引用添加到页面

	<!DOCTYPE html>
	<html lang="en">
	<head>
	  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
	  <meta charset="utf-8" />
	  <link rel="icon" href="<%= BASE_URL %>favicon.ico">
	  <title>Login Page - Ace Admin</title>
	
	  <meta name="description" content="User login page" />
	  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
	
	  <!-- bootstrap & fontawesome -->
	  <link rel="stylesheet" href="<%= BASE_URL %>ace/assets/css/bootstrap.min.css" />
	  <link rel="stylesheet" href="<%= BASE_URL %>ace/assets/font-awesome/4.5.0/css/font-awesome.min.css" />
	
	  <!-- text fonts -->
	  <link rel="stylesheet" href="<%= BASE_URL %>ace/assets/css/fonts.googleapis.com.css" />
	
	  <!-- ace styles -->
	  <link rel="stylesheet" href="<%= BASE_URL %>ace/assets/css/ace.min.css" />
	
	  <!--[if lte IE 9]>
	  <link rel="stylesheet" href="<%= BASE_URL %>ace/assets/css/ace-part2.min.css" />
	  <![endif]-->
	  <link rel="stylesheet" href="<%= BASE_URL %>ace/assets/css/ace-rtl.min.css" />
	
	  <!--[if lte IE 9]>
	  <link rel="stylesheet" href="<%= BASE_URL %>ace/assets/css/ace-ie.min.css" />
	  <![endif]-->
	
	  <!-- HTML5shiv and Respond.js for IE8 to support HTML5 elements and media queries -->
	
	  <!--[if lte IE 8]>
	  <script src="<%= BASE_URL %>ace/assets/js/html5shiv.min.js"></script>
	  <script src="<%= BASE_URL %>ace/assets/js/respond.min.js"></script>
	  <![endif]-->
	
	  <!--[if !IE]> -->
	  <script src="<%= BASE_URL %>ace/assets/js/jquery-2.1.4.min.js"></script>
	
	  <!-- <![endif]-->
	
	  <!--[if IE]>
	  <script src="<%= BASE_URL %>ace/assets/js/jquery-1.11.3.min.js"></script>
	  <![endif]-->
	  <script type="text/javascript">
	    if('ontouchstart' in document.documentElement) document.write("<script src='<%= BASE_URL %>ace/assets/js/jquery.mobile.custom.min.js'>"+"<"+"/script>");
	  </script>
	</head>
	  <body>
	    <noscript>
	      <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
	    </noscript>
	    <div id="app"></div>
	    <!-- built files will be auto injected -->
	  </body>
	</html>

修改App.vue,将Ace模板中的login.html代码主题部分移入

    <details>
    <summary>查看代码</summary>
    <pre><code>  
    &nbsp;
    这里写需要被折叠的代码
    &nbsp;
    </code></pre>
    </details>

