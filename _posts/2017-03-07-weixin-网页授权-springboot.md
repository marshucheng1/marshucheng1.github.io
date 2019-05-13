---
layout: post
title: 微信网页授权-springboot
date: 2017-03-07 12:31:22
categories: JavaWeb
tags: weixin
author: MarsHu
---

* content
{:toc}

### 微信网页授权 ###
在不使用第三方SDK的前提下，按照微信公众号开发文档，一步步获取`openid`。这里我们主要通过三个步骤来实现。

第一步：设置域名

第二步：获取code

第三步：获取access_token

这里官方文档中主要有2中获取方式，应对不同的业务场景。【关于网页授权的两种scope的区别说明】。这里先不用深究，使用过程中，自会明白。






### 设置域名 ###
设置域名的意思是，你需要将你的业务访问域名配置到微信公众号的`网页授权域名`中。通过如下操作可以看到。

登录微信公众平台->设置->公众号设置->功能设置->网页授权域名->设置

这里会有一个txt文件下载。将txt文件下载并放到你的项目页面访问路径的根目录下。这里如果你的项目不是根路径访问的话，需要配置下，直接访问域名能够通过验证则完成。

假设我这里配置的域名是:`https://www.test.com`

### 获取code ###
在后台中编写`controller`。

	package com.study.controller;
	
	import lombok.extern.slf4j.Slf4j;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.RestController;
	import org.springframework.web.client.RestTemplate;
	
	/**
	 * 手动获取用户openid
	 */
	@RestController
	@RequestMapping("/weixin")
	@Slf4j
	public class WeixinController {
	
	    @GetMapping("/auth")
	    public void auth() {
	        log.info("进入auth方法");
	    }
	}

修改示例访问请求：

`https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect`

这里需要修改成你的相关配置。主要参数有：
`appid`、`redirect_uri`、`reponse_type`、`scope`、`state`、`wechat_redirect`

至于参数的含义及用途参考官方文档。

修改后的访问请求如下:

`https://open.weixin.qq.com/connect/oauth2/authorize?appid=你的微信公众号AppID&redirect_uri=https://www.test.com/sell/weixin/auth&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect`

这里如果点击该访问请求后。会跳转到到`redirect_uri`的地址，格式如下。`redirect_uri/?code=CODE&state=STATE`。所以我们要对我们的`controller`进行修改，拿到回调后的`code`

	package com.study.controller;
	
	import lombok.extern.slf4j.Slf4j;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.RestController;
	import org.springframework.web.client.RestTemplate;
	
	/**
	 * 手动获取用户openid
	 */
	@RestController
	@RequestMapping("/weixin")
	@Slf4j
	public class WeixinController {
	
	    @GetMapping("/auth")
	    public void auth(@RequestParam("code") String code) {
	        log.info("进入auth方法");
	        log.info("code={}", code);
	    }
	}

现在我们可以在微信里访问这个地址，然后观察控制台的输出。一切正常的情况下，就能看到`code`了。

### 获取access_token ###
因为code具有时效性，所以我们需要进一步获取`access_token`。这里文档中也给出了示例请求：

`https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code`

修改为自己的相关配置。主要参数有：`appid`、`secret`、`code`、`grant_type`。相关参数的含义请参考官方文档。

修改后的请求：

`https://api.weixin.qq.com/sns/oauth2/access_token?appid=你的微信公众号AppID&secret=你的微信公众号AppSecret&code=" + code + "&grant_type=authorization_code`

这个`code`就是在第二步中获取的`code`

所以我们需要修改`controller`，来获取`access_token`

	package com.study.controller;
	
	import lombok.extern.slf4j.Slf4j;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.RestController;
	import org.springframework.web.client.RestTemplate;
	
	/**
	 * 手动获取用户openid
	 */
	@RestController
	@RequestMapping("/weixin")
	@Slf4j
	public class WeixinController {
	
	    @GetMapping("/auth")
	    public void auth(@RequestParam("code") String code) {
	        log.info("进入auth方法");
	        log.info("code={}", code);
	
	        String url = "https://api.weixin.qq.com/sns/oauth2/access_token?appid=你的微信公众号AppID&secret=你的微信公众号AppSecret&code=" + code + "&grant_type=authorization_code";
	        /*RestTemplate这个东西可以看成是HttpClient或是其他一些http请求工具*/
	        RestTemplate restTemplate = new RestTemplate();
	        String response = restTemplate.getForObject(url, String.class);
	        log.info("response={}", response);
	    }
	}


### 拉取用户信息(需scope为 snsapi_userinfo) ###
通过前面2步，我们可以成功获取`access_token和openid`了。然后我们可以通过这2个参数获取用户的信息。

这里官方文档中也给出了示例请求：

`https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN`

这里的主要参数有：`access_token`、`openid`、`lang`。相关参数含义参考官方文档。

修改后的请求：

`"https://api.weixin.qq.com/sns/userinfo?access_token=" + accessToken + "&openid=" + openid + "&lang=zh_CN"`

同时我们需要修改`controller`，来获取`access_token`

	package com.study.controller;
	
	import com.google.gson.Gson;
	import com.google.gson.reflect.TypeToken;
	import lombok.extern.slf4j.Slf4j;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.RestController;
	import org.springframework.web.client.RestTemplate;
	
	import java.util.Map;
	
	/**
	 * 手动获取用户openid
	 */
	@RestController
	@RequestMapping("/weixin")
	@Slf4j
	public class WeixinController {
	
	    @GetMapping("/auth")
	    public void auth(@RequestParam("code") String code) {
	        log.info("进入auth方法");
	        log.info("code={}", code);
	
	        String url = "https://api.weixin.qq.com/sns/oauth2/access_token?appid=wxe0af2574c170f1d5&secret=08cca0004211cab7f7e2acbef037d2fc&code=" + code + "&grant_type=authorization_code";
	        /*RestTemplate这个东西可以看成是HttpClient或是其他一些http请求工具*/
	        RestTemplate restTemplate = new RestTemplate();
	        String response = restTemplate.getForObject(url, String.class);
	        log.info("response={}", response);
	
	        /*拉取用户信息使用*/
	        /*获取access_token、openid*/
	        String accessToken = getJsonArrayValue(response, "access_token");
	        String openid = getJsonArrayValue(response, "openid");
	        log.info("access_token={}", accessToken);
	        log.info("openid={}", openid);
	        /*访问请求*/
	        String userDetailUrl = "https://api.weixin.qq.com/sns/userinfo?access_token=" + accessToken + "&openid=" + openid + "&lang=zh_CN";
	        RestTemplate userTemplate = new RestTemplate();
	        String userDetail = userTemplate.getForObject(userDetailUrl, String.class);
	        log.info("userDetail={}", userDetail);
	
	    }
	
	    /*对JSON数据进行处理---这里与传统的处理方式有点区别*/
	    public String getJsonArrayValue(String response, String key){
	        String jsonValue = "";
	        if(response == null || response.trim().length() == 0){
	            return null;
	        }
	        Gson gson = new Gson();
	        Map<String, String> userDetail = gson.fromJson(response, new TypeToken<Map<String, String>>() {}.getType());
	        jsonValue = userDetail.get(key);
	        return jsonValue;
	    }
	
	}

后面的具体获取相应的数据，大家也可以通过该方法获取。

### 刷新access_token（如果需要） ###
这里就不演示这个了。有需要的可以自行思考。这里补充的是`scope`的值是`snsapi_base`时，一般是不弹窗获取`openid`。拿到的信息也是比较少的。这里官方文档中有详细的说明