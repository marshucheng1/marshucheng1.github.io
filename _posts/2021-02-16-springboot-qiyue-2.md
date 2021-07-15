---
layout: post
title: Springboot全局异常处理
date: 2021-02-16 12:10:22
categories: springboot
tags: springboot
author: MarsHu
---

* content
{:toc}

### 如何统一捕获异常  ###

	package com.zhqx.missyou.core;
	
	import org.springframework.web.bind.annotation.ControllerAdvice;
	import org.springframework.web.bind.annotation.ExceptionHandler;
	
	import javax.servlet.http.HttpServletRequest;
	
	@ControllerAdvice
	public class GlobalExceptionAdvice {
	
	    @ExceptionHandler(value = Exception.class)
	    public void handleException(HttpServletRequest request, Exception e) {
	        //处理逻辑
	    }
	}

### 异常分类Error、CheckedException与RunTimeException  ###

Error:错误,通常是系统级别的。Exception:异常

CheckedException:必须进行处理的异常,编译阶段不处理就会报错---通常是我们可以处理的异常

RuntimeException:运行时异常---通常是我们不可以处理的异常






### 同时监听Exception和HTTPException  ###

自定义HttpException;

	package com.zhqx.missyou.exception.http;
	
	public class HttpException extends RuntimeException {
	    protected Integer code;
	    protected Integer httpStatusCode = 500;
		//省略get和set方法...
	}

定义2个属于HttpException的子类异常

	package com.zhqx.missyou.exception.http;
	
	public class NotFoundException extends HttpException {
	    public NotFoundException(int code) {
	        this.code = code;
	        this.httpStatusCode = 404;
	    }
	}	

	package com.zhqx.missyou.exception.http;
	
	public class ForbiddenException extends HttpException {
	    public ForbiddenException(int code) {
	        this.code = code;
	        this.httpStatusCode = 403;
	    }
	}

修改异常处理类

	package com.zhqx.missyou.core;
	
	import com.zhqx.missyou.exception.http.HttpException;
	import org.springframework.web.bind.annotation.ControllerAdvice;
	import org.springframework.web.bind.annotation.ExceptionHandler;
	
	import javax.servlet.http.HttpServletRequest;
	
	@ControllerAdvice
	public class GlobalExceptionAdvice {
	
		//主要用来处理未知异常
	    @ExceptionHandler(value = Exception.class)
	    public void handleException(HttpServletRequest request, Exception e) {
	        //处理逻辑
	    }
		//主要用来处理已知异常
	    //当业务代码中抛出属于httpException的异常时,只会执行handleHttpException
	    @ExceptionHandler(HttpException.class)
	    public void handleHttpException(HttpServletRequest request, HttpException e) {
	        //处理httpException逻辑
	    }
	}

### 定义统一的异常返回信息类  ###
定义统一异常返回信息类UnifyResponse

	package com.zhqx.missyou.core;
	
	public class UnifyResponse {
	    private int code;
	    private String message;
	    private String request;
	
	    public UnifyResponse(int code, String message, String request) {
	        this.code = code;
	        this.message = message;
	        this.request = request;
	    }
	
	    //省略get、set方法
		//这里必须要有get方法,否则返回信息时,拿不到相关字段的值,会报错
	}

	修改异常处理类
	
	import com.zhqx.missyou.exception.http.HttpException;
	import org.springframework.web.bind.annotation.ControllerAdvice;
	import org.springframework.web.bind.annotation.ExceptionHandler;
	import org.springframework.web.bind.annotation.ResponseBody;
	
	import javax.servlet.http.HttpServletRequest;
	
	@ControllerAdvice
	public class GlobalExceptionAdvice {
	
	    @ExceptionHandler(value = Exception.class)
	    @ResponseBody
	    public UnifyResponse handleException(HttpServletRequest request, Exception e) {
	        UnifyResponse message = new UnifyResponse(9999, "服务器异常", "url");
	        return message;
	    }
	
	    //当业务代码中抛出属于httpException的异常时,只会执行handleHttpException
	    @ExceptionHandler(HttpException.class)
	    public void handleHttpException(HttpServletRequest request, HttpException e) {
	        //处理httpException逻辑
	    }
	}

进行如上修改后,我们在控制器中抛出Exception时,就会得到正常的json格式的返回信息。

	{
	    "code": 9999,
	    "message": "服务器异常",
	    "request": "url"
	}

但是这个时候,我们发现Status还是200,这是不符合常理的。因为我们系统已经出错了。使用@ResponseStatus注解来控制请求状态码。

	import com.zhqx.missyou.exception.http.HttpException;
	import org.springframework.http.HttpStatus;
	import org.springframework.web.bind.annotation.ControllerAdvice;
	import org.springframework.web.bind.annotation.ExceptionHandler;
	import org.springframework.web.bind.annotation.ResponseBody;
	import org.springframework.web.bind.annotation.ResponseStatus;
	
	import javax.servlet.http.HttpServletRequest;
	
	@ControllerAdvice
	public class GlobalExceptionAdvice {
	
	    @ExceptionHandler(value = Exception.class)
	    @ResponseBody
	    @ResponseStatus(code = HttpStatus.INTERNAL_SERVER_ERROR)
	    public UnifyResponse handleException(HttpServletRequest request, Exception e) {
	        String requestUrl = request.getRequestURI();
	        String method = request.getMethod();
			//打印异常信息
			System.out.println(e);
	        UnifyResponse message = new UnifyResponse(9999, "服务器异常", method + " " +requestUrl);
	        return message;
	    }
	
	    //当业务代码中抛出属于httpException的异常时,只会执行handleHttpException
	    @ExceptionHandler(HttpException.class)
	    public void handleHttpException(HttpServletRequest request, HttpException e) {
	        //处理httpException逻辑
	    }
	}

使用@ResponseStatus注解没有办法灵活的自定义状态码。当我们需要在控制器中抛出自定义的HttpException时,我们通常会自定义一个code值。

假设控制器中抛出的异常如下：

	import com.zhqx.missyou.exception.http.ForbiddenException;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	@RestController
	@RequestMapping("/v1/banner")
	public class BannerController {
	
	    @GetMapping("/test")
	    public String test() throws Exception {
	        throw new ForbiddenException(10001);
	    }
	}

这个时候,我们修改统一异常处理,针对自定义的HttpException做出修改。

	@ControllerAdvice
	public class GlobalExceptionAdvice {
	
	    @ExceptionHandler(value = Exception.class)
	    @ResponseBody
	    @ResponseStatus(code = HttpStatus.INTERNAL_SERVER_ERROR)
	    public UnifyResponse handleException(HttpServletRequest request, Exception e) {
	        String requestUrl = request.getRequestURI();
	        String method = request.getMethod();
			//打印异常信息
			System.out.println(e);
	        UnifyResponse message = new UnifyResponse(9999, "服务器异常", method + " " +requestUrl);
	        return message;
	    }
	
	    //当业务代码中抛出属于httpException的异常时,只会执行handleHttpException
	    @ExceptionHandler(HttpException.class)
	    public ResponseEntity<UnifyResponse> handleHttpException(HttpServletRequest request, HttpException e) {
	        //处理httpException逻辑
	        String requestUrl = request.getRequestURI();
	        String method = request.getMethod();
	        UnifyResponse message = new UnifyResponse(e.getCode(), "xxx", method + " " +requestUrl);
	        HttpHeaders headers = new HttpHeaders();
	        headers.setContentType(MediaType.APPLICATION_JSON);
	        HttpStatus status = HttpStatus.resolve(e.getHttpStatusCode());
	        ResponseEntity<UnifyResponse> responseEntity = new ResponseEntity<>(message, headers, status);
	        return responseEntity;
	    }
	}

### 定义异常信息的配置文件  ###
在上面的过程中,我们没有针对10001这个code码返回具体的错误信息。在很多时候我们会采取硬编码的方式,来返回错误信息。实际上,为了方便管理
错误信息。我们也可以将所有错误信息保存在配置文件中。

定义错误信息配置文件。文件位置resources/config/exception-code.properties.

	zhqx.codes[10000]="通用异常"
	zhqx.codes[10001]="通用参数异常"

定义错误信息配置类,与配置文件对应

	package com.zhqx.missyou.core.configuration;
	
	import java.util.HashMap;
	import java.util.Map;

	//对应配置文件中的前缀
	@ConfigurationProperties(prefix = "zhqx")
	//读取对应的配置文件
	@PropertySource("classpath:config/exception-code.properties")
	//交由容器管理
	@Component
	public class ExceptionCodeConfiguration {
		//这里需要codes与配置文件中的一致
	    private Map<Integer, String> codes = new HashMap<>();
	
	    public Map<Integer, String> getCodes() {
	        return codes;
	    }
	
	    public void setCodes(Map<Integer, String> codes) {
	        this.codes = codes;
	    }
	
	    public String getMessage(int code) {
	        String message = codes.get(code);
	        return message;
	    }
	}

修改统一异常处理类：

	package com.zhqx.missyou.core;
	
	import com.zhqx.missyou.core.configuration.ExceptionCodeConfiguration;
	import com.zhqx.missyou.exception.http.HttpException;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.http.HttpHeaders;
	import org.springframework.http.HttpStatus;
	import org.springframework.http.MediaType;
	import org.springframework.http.ResponseEntity;
	import org.springframework.web.bind.annotation.ControllerAdvice;
	import org.springframework.web.bind.annotation.ExceptionHandler;
	import org.springframework.web.bind.annotation.ResponseBody;
	import org.springframework.web.bind.annotation.ResponseStatus;
	
	import javax.servlet.http.HttpServletRequest;
	
	@ControllerAdvice
	public class GlobalExceptionAdvice {
	    
		//关联异常信息配置类
	    @Autowired
	    private ExceptionCodeConfiguration exceptionCodeConfiguration;
	
	    @ExceptionHandler(value = Exception.class)
	    @ResponseBody
	    @ResponseStatus(code = HttpStatus.INTERNAL_SERVER_ERROR)
	    public UnifyResponse handleException(HttpServletRequest request, Exception e) {
	        String requestUrl = request.getRequestURI();
	        String method = request.getMethod();
			//打印异常信息
			System.out.println(e);
	        UnifyResponse message = new UnifyResponse(9999, "服务器异常", method + " " +requestUrl);
	        return message;
	    }
	
	    //当业务代码中抛出属于httpException的异常时,只会执行handleHttpException
	    @ExceptionHandler(HttpException.class)
	    public ResponseEntity<UnifyResponse> handleHttpException(HttpServletRequest request, HttpException e) {
	        //处理httpException逻辑
	        String requestUrl = request.getRequestURI();
	        String method = request.getMethod();
			//读取对应code值的message信息 
	        UnifyResponse message = new UnifyResponse(e.getCode(), exceptionCodeConfiguration.getMessage(e.getCode()), method + " " +requestUrl);
	        HttpHeaders headers = new HttpHeaders();
	        headers.setContentType(MediaType.APPLICATION_JSON);
	        HttpStatus status = HttpStatus.resolve(e.getHttpStatusCode());
	        ResponseEntity<UnifyResponse> responseEntity = new ResponseEntity<>(message, headers, status);
	        return responseEntity;
	    }
	}

这个时候,我们重新访问,我们得到返回结果

	{
	    "code": 10001,
	    "message": "\"éç¨åæ°å¼å¸¸\"",
	    "request": "GET /v1/banner/test"
	}

这里会有2个问题,一个是返回信息出现乱码,第二个是,返回的信息加上了引号 ""。

乱码需要修改File | Settings | Editor | File Encodings配置项的properties file的编码为UTF-8,并勾选后面的native-to-ascii选项

引号问题需要将原来配置文件中的引号去掉
	
	zhqx.codes[10000]=通用异常
	zhqx.codes[10001]=通用参数异常

再次访问后得到正常信息

	{
	    "code": 10001,
	    "message": "通用参数异常",
	    "request": "GET /v1/banner/test"
	}


### 根据目录结构,自动添加请求的前缀  ###
当前我们控制器的路径结构是com.zhqx.missyou.api.v1,如下

	package com.zhqx.missyou.api.v1;
	
	import com.zhqx.missyou.exception.http.ForbiddenException;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	@RestController
	@RequestMapping("/v1/banner")
	public class BannerController {
	
	    @GetMapping("/test")
	    public String test() throws Exception {
	        throw new ForbiddenException(10001);
	    }
	}

当我们在v1目录下有很多个控制器的时候,我们如何避免在每个控制器上都手动书写/v1这样的前缀呢。

Springboot默认情况下是通过RequestMappingInfo来统一处理@RequestMapping注解的请求。

所以我们需要自定义一个RequestMappingInfo,实现添加请求前缀

	package com.zhqx.missyou.core.hack;
	
	import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
	import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
	
	import java.lang.reflect.Method;
	
	public class AutoPrefixUrlMapping extends RequestMappingHandlerMapping {
	
	    @Override
	    protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
	        RequestMappingInfo requestMappingInfo = super.getMappingForMethod(method, handlerType);
	        if(requestMappingInfo != null) {
	
	        }
	        return requestMappingInfo;
	    }
	
	    private String getPrefix(Class<?> handlerType) {
	        String packageName = handlerType.getPackage().getName();
	        return packageName;
	    }
	}

RequestMappingInfo自定义之后,我们还需要将他注册到整个容器中,我们需要定义一个配置类,让Springboot处理请求时,会调用我们自定义请求处理类

	package com.zhqx.missyou.core.configuration;
	
	import com.zhqx.missyou.core.hack.AutoPrefixUrlMapping;
	import org.springframework.boot.autoconfigure.web.servlet.WebMvcRegistrations;
	import org.springframework.stereotype.Component;
	import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
	
	@Component
	public class AutoPrefixConfiguration implements WebMvcRegistrations {
	    @Override
	    public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
	        return new AutoPrefixUrlMapping();
	    }
	}

在配置文件application.properties中定义当前所有控制器所处的根目录
		
	demo.condition=demo1
	zhqx.api-package = com.zhqx.missyou.api

修改自定义的RequestMappingInfo实现类

	package com.zhqx.missyou.core.hack;
	
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
	import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
	
	import java.lang.reflect.Method;
	
	public class AutoPrefixUrlMapping extends RequestMappingHandlerMapping {
	
	    //读取配置文件中自定义的控制器所处根路径
	    @Value("${zhqx.api-package}")
	    private String apiPackagePath;
	
	    @Override
	    protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
	        RequestMappingInfo requestMappingInfo = super.getMappingForMethod(method, handlerType);
	        if(requestMappingInfo != null) {
	            String prefix = this.getPrefix(handlerType);
	            return RequestMappingInfo.paths(prefix).build().combine(requestMappingInfo);
	        }
	        return requestMappingInfo;
	    }
	
	    private String getPrefix(Class<?> handlerType) {
	        String packageName = handlerType.getPackage().getName();
	        String dotPath = packageName.replaceAll(this.apiPackagePath, "");
	        return dotPath.replace(".", "/");
	    }
	}

这个时候我们修改控制器的请求

	package com.zhqx.missyou.api.v1;
	
	import com.zhqx.missyou.exception.http.ForbiddenException;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	@RestController
	@RequestMapping("/banner")
	public class BannerController {
	
	    @GetMapping("/test")
	    public String test() throws Exception {
	        throw new ForbiddenException(10001);
	    }
	}

我们仍然可以通过http://localhost:8080/v1/banner/test来访问请求。

当我们的控制器位于com.zhqx.missyou.api.v1.other包下时

	package com.zhqx.missyou.api.v1.other;
	
	import com.zhqx.missyou.exception.http.ForbiddenException;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	@RestController
	@RequestMapping("/banner")
	public class BannerController {
	
	    @GetMapping("/test")
	    public String test() throws Exception {
	        throw new ForbiddenException(10001);
	    }
	}


我们可以通过http://localhost:8080/v1/other/banner/test来访问请求。