---
layout: post
title: Springboot请求参数获取和验证
date: 2021-02-17 12:10:22
categories: springboot
tags: springboot
author: MarsHu
---

* content
{:toc}

### 请求中参数值,?号后参数值获取  ###
当前请求为http://localhost:8080/v1/banner/test/1?name=tom。如何在控制器中获取路径的参数值1,以及参数name的值

使用@PathVariable注解来获取请求地址中参数值,使用@RequestParam来获请求地址取携带的参数值。当携带的参数名与方法参数名一致时,可以省略

	package com.zhqx.missyou.api.v1;
	
	import com.zhqx.missyou.exception.http.ForbiddenException;
	import org.springframework.web.bind.annotation.*;
	
	@RestController
	@RequestMapping("/banner")
	public class BannerController {
	
	    @GetMapping("/test/{id}")
	    public String test(@PathVariable Integer id,@RequestParam String name) throws Exception {
	        throw new ForbiddenException(10001);
	    }
	}






如果时post请求,且前端提交的参数是json类型,后端如何接收。前端请求参数如下。

	{"name":"tome","age":17}

如果没有特别的实体来封装对应的参数。我们可以使用Map<String, Object>来接收参数

    @PostMapping("/test/{id}")
    public String test1(@PathVariable Integer id, @RequestBody Map<String, Object> person) throws Exception {
        throw new ForbiddenException(10000);
    }

我们也可以定义数据传输对象PersonDTO,用它来封装参数值

	package com.zhqx.missyou.dto;
	
	public class PersonDTO {
	    private String name;
	    private Integer age;
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    public Integer getAge() {
	        return age;
	    }
	
	    public void setAge(Integer age) {
	        this.age = age;
	    }
	}

	//控制器代码
    @PostMapping("/test/{id}")
    public String test1(@PathVariable Integer id, @RequestBody PersonDTO personDTO) throws Exception {
        throw new ForbiddenException(10000);
    }



### 使用@Validated注解进行基础参数校验  ###
Springboot2.3版本后,web模块下面没有依赖 validation。需要在pom文件中引入校验包。

	<dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

假设前端请求为:http://localhost:8080/v1/banner/test/1

后端的控制器逻辑为：

	@PostMapping("/test/{id}")
    public String test1(@PathVariable Integer id, @RequestBody PersonDTO personDTO) throws Exception {
        return "ok";
    }

我们可以通过简单的校验注解来控制id的范围。使用@Range注解

	@PostMapping("/test/{id}")
    public String test1(@PathVariable @Range(min = 1, max = 10, message = "id范围为1-10") Integer id, @RequestBody PersonDTO personDTO) throws Exception {
        return "ok";
    }

此时我们前端请求为：http://localhost:8080/v1/banner/test/11,这个时候我们会发现请求正常,并没有出错。

当我们在使用校验注解时,我们需要在控制器上添加@Validated注解,来开启对校验注解的支持。

	@Validated
	public class BannerController {
		//...
	}

添加后,我们访问后,发现服务器返回如下错误提示,但是控制台并没有任何关于校验的信息。

	{
	    "code": 9999,
	    "message": "服务器异常",
	    "request": "POST /v1/banner/test/11"
	}

这是因为我们对异常信息进行了统一处理,捕获的异常信息直接被覆盖了。修改统一异常信息处理类

	//注释掉,当前的异常处理逻辑
	//@ExceptionHandler(value = Exception.class)
    @ResponseBody
    @ResponseStatus(code = HttpStatus.INTERNAL_SERVER_ERROR)
    public UnifyResponse handleException(HttpServletRequest request, Exception e) {
        String requestUrl = request.getRequestURI();
        String method = request.getMethod();
        UnifyResponse message = new UnifyResponse(9999, "服务器异常", method + " " +requestUrl);
        return message;
    }

注释掉之后,再次访问,控制台会打印错误信息:

	javax.validation.ConstraintViolationException: test1.id: id范围为1-10

### 使用@Validated注解进行对象参数校验  ###
继续保留上一步对统一异常处理类的修改,即注释掉@ExceptionHandler(value = Exception.class)。

当我们对PersonDTO的name属性进行参数验证

	public class PersonDTO {
	    @Length(min = 1, max = 4, message = "名字为1-4个字符")
	    private String name;
		//...省略其他代码
	}

前端请求为:http://localhost:8080/v1/banner/test/10,[post方式]。携带参数如下:

	{"name":"tomcat","age":17}

后端控制器为:

	@PostMapping("/test/{id}")
    public String test1(@PathVariable @Range(min = 1, max = 10, message = "id范围为1-10") Integer id,
                        @RequestBody PersonDTO personDTO) throws Exception {
        //throw new ForbiddenException(10000);
        return "ok";
    }

我们会发现控制台并没有出错。这是因为实体参数的校验,需要在实体参数上使用@Validated注解,修改控制器如下:

	@PostMapping("/test/{id}")
    public String test1(@PathVariable @Range(min = 1, max = 10, message = "id范围为1-10") Integer id,
                        @RequestBody @Validated PersonDTO personDTO) throws Exception {
        //throw new ForbiddenException(10000);
        return "ok";
    }

再次访问请求时,我们会发现控制台会打印出错误信息:

	DefaultMessageSourceResolvable: codes [personDTO.name,name]; arguments []; default message [name],4,1]; default message [名字为1-4个字符]] ]


### 级联对象参数校验  ###
继续保留上一步对统一异常处理类的修改,即注释掉@ExceptionHandler(value = Exception.class)。

修改PersonDTO,追加SchoolDTO对象,假设我们需要校验SchoolDTO对象的schoolName属性。

	package com.zhqx.missyou.dto;
	
	public class SchoolDTO {
		@Length(min = 2, message = "学校名称最少2个字符")
	    private String schoolName;
	
	    //...省略其他代码
	}
	

	public class PersonDTO {
	    @Length(min = 1, max = 4, message = "名字为1-4个字符")
	    private String name;
	
	    private Integer age;
	
	    private SchoolDTO schoolDTO;
		
		//...省略其他代码
	}

前端请求为:http://localhost:8080/v1/banner/test/10,[post方式]。携带参数如下:

	{"name":"tom","age":17,"schoolDTO":{"schoolName":"q"}}

此时访问控制器,发现并没有校验schoolDTO的schoolName属性。

当我们需要校验级联对象的属性时,我们需要在校验对象上加上@Valid注解

	public class PersonDTO {
	    @Length(min = 1, max = 4, message = "名字为1-4个字符")
	    private String name;
	
	    private Integer age;
	
		@Valid
	    private SchoolDTO schoolDTO;
		
		//...省略其他代码
	}

再次访问请求,这时候控制台即能打印出错误信息

	default message [schoolDTO.schoolName],2147483647,2]; default message [学校名称最少2个字符]] ]


### 自定义校验  ###
基础的校验,能够实现一些基础校验.实际开发中,我们需要自定义注解来帮助我们实现参数校验。修改PersonDTO,去除SchoolDTO

	public class PersonDTO {
	    @Length(min = 1, max = 4, message = "名字为1-4个字符")
	    private String name;
	
	    private Integer age;
	
	    private String password1;
	
	    private String password2;
		
		//...省略其他代码
	
	}

假设我们需要校验password1和password2的值是一致的。这个时候我们就需要使用自定义注解。

创建自定义注解PasswordEqual

	package com.zhqx.missyou.validators;
	
	import javax.validation.Payload;
	import java.lang.annotation.*;
	
	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target({ElementType.TYPE})
	public @interface PasswordEqual {
	    String message() default "";
	
	    Class<?>[] groups() default {};
	
	    Class<? extends Payload>[] payLoad() default {};
	}

创建自定义注解PasswordEqual的逻辑处理类PasswordValidator

	package com.zhqx.missyou.validators;
	
	import com.zhqx.missyou.dto.PersonDTO;
	
	import javax.validation.ConstraintValidator;
	import javax.validation.ConstraintValidatorContext;
	
	public class PasswordValidator implements ConstraintValidator<PasswordEqual, PersonDTO> {
	    @Override
	    public boolean isValid(PersonDTO personDTO, ConstraintValidatorContext constraintValidatorContext) {
	        String password1 = personDTO.getPassword1();
        	String password2 = personDTO.getPassword2();
        	boolean match = false;
        	if(password1 != null && password2 != null) {
            	match = password1.equals(password2);
        	}
        	return match;
	    }
	}

将PasswordEqual和PasswordValidator进行关联

	package com.zhqx.missyou.validators;
	
	import javax.validation.Constraint;
	import javax.validation.Payload;
	import java.lang.annotation.*;
	
	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target({ElementType.TYPE})
	@Constraint(validatedBy = PasswordValidator.class)
	public @interface PasswordEqual {
	    String message() default "";
	
	    Class<?>[] groups() default {};
	
	    Class<? extends Payload>[] payload() default {};
	}

使用自定义注解

	@PasswordEqual
	public class PersonDTO {
	    @Length(min = 1, max = 4, message = "名字为1-4个字符")
	    private String name;
	
	    private Integer age;
	
	    private String password1;
	
	    private String password2;
		
		//...省略其他代码
	
	}

继续保留上一步对统一异常处理类的修改,即注释掉@ExceptionHandler(value = Exception.class)。

前端访问:http://localhost:8080/v1/banner/test/10,post请求,携带参数如下:

	{"name":"tom","age":17,"password1":"123456","password2":"12345"}

访问控制器为:

	@PostMapping("/test/{id}")
    public String test1(@PathVariable @Range(min = 1, max = 10, message = "id范围为1-10") Integer id,
                        @RequestBody @Validated PersonDTO personDTO) throws Exception {
        //throw new ForbiddenException(10000);
        return "ok";
    }

控制台打印错误信息

	codes [personDTO.,]; arguments []; default message []]; default message [passwords are not equal]] ]

假设我们还需在注解中添加额外的参数,比如控制密码的最小长度为4和最大长度为6

	package com.zhqx.missyou.validators;
	
	import javax.validation.Constraint;
	import javax.validation.Payload;
	import java.lang.annotation.*;
	
	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target({ElementType.TYPE})
	@Constraint(validatedBy = PasswordValidator.class)
	public @interface PasswordEqual {
	    int min() default 4;
	
	    int max() default 6;
	
	    String message() default "passwords are not equal";
	
	    Class<?>[] groups() default {};
	
	    Class<? extends Payload>[] payload() default {};
	}

修改默认的密码长度限制

@PasswordEqual(min = 1)
public class PersonDTO {
    @Length(min = 1, max = 4, message = "名字为1-4个字符")
    private String name;

    private Integer age;

    private String password1;

    private String password2;、

	//...省略其他代码

}

修改注解处理类PasswordValidator

	package com.zhqx.missyou.validators;
	
	import com.zhqx.missyou.dto.PersonDTO;
	
	import javax.validation.ConstraintValidator;
	import javax.validation.ConstraintValidatorContext;
	
	public class PasswordValidator implements ConstraintValidator<PasswordEqual, PersonDTO> {
	    private int min;
	    private int max;
	
	    @Override
	    public void initialize(PasswordEqual constraintAnnotation) {
	        this.min = constraintAnnotation.min();
	        this.max = constraintAnnotation.max();
	    }
	
	    @Override
	    public boolean isValid(PersonDTO personDTO, ConstraintValidatorContext constraintValidatorContext) {
	        String password1 = personDTO.getPassword1();
	        String password2 = personDTO.getPassword2();
	        boolean match = false;
	        if(password1 != null && password2 != null) {
	            match = password1.equals(password2);
	
	            if(password1.length() < min) {
	                //省略逻辑代码
	            }
	            if(password1.length() > max) {
	                //省略逻辑代码
	            }
	            if(password2.length() < min) {
	                //省略逻辑代码
	            }
	            if(password2.length() > max) {
	                //省略逻辑代码
	            }
	        }
	        return match;
	    }
	}

DTO参数校验异常属于MethodArgumentNotValidException,所以我们可以在GlobalExceptionAdvice新增一个统一异常处理

		//DTO参数校验异常属于MethodArgumentNotValidException
	    @ExceptionHandler(MethodArgumentNotValidException.class)
	    @ResponseBody
	    @ResponseStatus(code = HttpStatus.BAD_REQUEST)//参数错误状态码
	    public UnifyResponse handleBeanValidation(HttpServletRequest request, MethodArgumentNotValidException e) {
	        String requestUrl = request.getRequestURI();
	        String method = request.getMethod();
	
	        List<ObjectError> errors = e.getBindingResult().getAllErrors();
	        String message = this.formatAllErrorMessages(errors);
	
	        return new UnifyResponse(10001, message, method + " " +requestUrl);
	    }
	
	    private String formatAllErrorMessages(List<ObjectError> errors) {
	        StringBuffer errorMsg = new StringBuffer();
	        errors.forEach(error -> errorMsg.append(error.getDefaultMessage()).append(";"));
	        return errorMsg.toString();
	    }

这样,当PersonDTO中的参数出现验证错误时,会执行此方法。

请求地址中的参数校验异常属于ConstraintViolationException,所以我们可以在GlobalExceptionAdvice新增一个统一异常处理

	//请求地址中的参数校验异常属于ConstraintViolationException
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseBody
    @ResponseStatus(code = HttpStatus.BAD_REQUEST)//参数错误状态码
    public UnifyResponse handleConstraintValidation(HttpServletRequest request, ConstraintViolationException e) {
        String requestUrl = request.getRequestURI();
        String method = request.getMethod();
        String message = e.getMessage();

        return new UnifyResponse(10001, message, method + " " +requestUrl);
    }

这样,当请求地址中的参数出现验证错误时,会执行此方法。

需要注意的是PersonDTO中的参数校验异常是高于请求地址中的参数校验异常的。当二者同时出现异常时,会有限处理PersonDTO中的参数校验异常。