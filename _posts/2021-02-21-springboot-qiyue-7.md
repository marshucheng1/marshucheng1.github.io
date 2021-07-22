---
layout: post
title: Springboot-JWT令牌
date: 2021-02-21 12:10:22
categories: springboot
tags: springboot
author: MarsHu
---

* content
{:toc}

### TokenGetDTO--基于jwt登录校验实体  ###

> **1.TokenGetDTO:**

	package com.zhqx.missyou.dto;
	
	import com.zhqx.missyou.core.enumeration.LoginType;
	import com.zhqx.missyou.dto.validators.TokenPassword;
	import lombok.Getter;
	import lombok.Setter;
	
	import javax.validation.constraints.NotBlank;
	
	/**
	 * 通用用户登录dto
	 * 在不同登录方式中
	 * account和password代表的内容不同
	 */
	@Getter
	@Setter
	public class TokenGetDTO {
	    @NotBlank(message = "account不允许为空")
	    private String account;
	    @TokenPassword(max=30, message = "{token.password}")
	    private String password;
	
	    private LoginType type;
	}








> **2.@TokenPassword,自定义密码校验注解:**

TokenPassword:

	package com.zhqx.missyou.dto.validators;
	
	import javax.validation.Constraint;
	import javax.validation.Payload;
	import java.lang.annotation.*;
	
	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target({ElementType.TYPE, ElementType.FIELD})
	@Constraint(validatedBy = TokenPasswordValidator.class )
	public @interface TokenPassword {
	    String message() default "字段不符合要求";
	
	    int min() default 6;
	
	    int max() default 32;
	
	    Class<?>[] groups() default {};
	
	    Class<? extends Payload>[] payload() default {};
	}

TokenPasswordValidator:

	package com.zhqx.missyou.dto.validators;
	
	import org.springframework.util.StringUtils;
	
	import javax.validation.ConstraintValidator;
	import javax.validation.ConstraintValidatorContext;
	
	public class TokenPasswordValidator implements ConstraintValidator<TokenPassword, String> {
	    private Integer min;
	    private Integer max;
	
	    @Override
	    public void initialize(TokenPassword constraintAnnotation) {
	        this.min = constraintAnnotation.min();
	        this.max = constraintAnnotation.max();
	    }
	
	    @Override
	    public boolean isValid(String s, ConstraintValidatorContext constraintValidatorContext) {
			//StringUtils.isEmpty() 2.5.2废弃 -- 使用 !StringUtils.hasText(s)做条件
	        if(StringUtils.isEmpty(s)){
				//小程序登录不需要password,所以直接返回true
	            return true;
	        }
	        return s.length() >= this.min && s.length() <= this.max;
	    }
	}

> **3.LoginType,登录方式枚举:**

	package com.zhqx.missyou.core.enumeration;
	
	public enum  LoginType {
	    USER_WX(0, "微信登录"),
	    USER_EMAIL(1, "邮箱登录");
	
	    private Integer value;
	
	    LoginType(Integer value, String description) {
	        this.value = value;
	    }
	}

在resources目录下的ValidationMessages.properties配置密码校验信息:

	id.positive = id必须是正整数
	token.password = password不符合规范：当前值是${validatedValue}；最大值应该是{max}，最小值应该是{min}

### 获取微信小程序登录用户  ###
微信小程序调用wx.login方法.

    wx.login({
      success: (res) => {
        if (res.code) {
          wx.request({
            url: 'http://localhost:8080/v1/token',
            method: 'POST',
            data: {
              account: res.code,
              type: 0
            },
            success: (res) => {
              console.log(res.data)
              const code = res.statusCode.toString()
              if (code.startsWith('2')) {
                wx.setStorageSync('token', res.data.token)
              }
            }
          })
        }
      }
    })

在配置文件application.yml配置微信appid等信息:

	wx:
	  appid: xxx
	  appsecret: xxx
	  code2session: https://api.weixin.qq.com/sns/jscode2session?appid={0}&secret={1}&js_code={2}&grant_type=authorization_code


WxAuthenticationServiceImpl类:

	package com.zhqx.missyou.service.impl;
	
	import com.fasterxml.jackson.core.JsonProcessingException;
	import com.fasterxml.jackson.databind.ObjectMapper;
	import com.zhqx.missyou.service.WxAuthenticationService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.stereotype.Service;
	import org.springframework.web.client.RestTemplate;
	
	import java.text.MessageFormat;
	import java.util.HashMap;
	import java.util.Map;
	
	@Service
	public class WxAuthenticationServiceImpl implements WxAuthenticationService {
	
	    @Autowired
	    private ObjectMapper mapper;
	
	    @Value("${wx.code2session}")
	    private String code2SessionUrl;
	    @Value("${wx.appid}")
	    private String appid;
	    @Value("${wx.appsecret}")
	    private String appsecret;
	
	    @Override
	    public String code2Session(String code) {
	        String url = MessageFormat.format(this.code2SessionUrl, this.appid, this.appsecret, code);
	        RestTemplate rest = new RestTemplate();
	        Map<String, Object> session = new HashMap<>();
	        String sessionText = rest.getForObject(url, String.class);
	        try {
	            session = mapper.readValue(sessionText, Map.class);
	        } catch (JsonProcessingException e) {
	            e.printStackTrace();
	        }
	        return this.registerUser(session);
	    }
	
		@Override
    	public String registerUser(Map<String, Object> session) {
        	return null;
    	}
	
	}

TokenController类:

	package com.zhqx.missyou.api.v1;
	
	import com.zhqx.missyou.dto.TokenGetDTO;
	import com.zhqx.missyou.exception.http.NotFoundException;
	import com.zhqx.missyou.service.WxAuthenticationService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.validation.annotation.Validated;
	import org.springframework.web.bind.annotation.PostMapping;
	import org.springframework.web.bind.annotation.RequestBody;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	import java.util.HashMap;
	import java.util.Map;
	
	@RequestMapping(value = "token")
	@RestController
	public class TokenController {
	
	    @Autowired
	    private WxAuthenticationService wxAuthenticationService;
	
	    @PostMapping("")
	    public Map<String, String> getToken(@RequestBody @Validated TokenGetDTO userData) {
	        Map<String, String> map = new HashMap<>();
	        String token = null;
	        switch (userData.getType()) {
	            case USER_WX:
	                token = wxAuthenticationService.code2Session(userData.getAccount());
	                break;
	            case USER_EMAIL:
	                break;
	            default:
	                throw new NotFoundException(10003);
	        }
	        map.put("token", token);
	        return map;
	    }
	}

### 注册小程序用户或发放jwt  ###
User类

	package com.zhqx.missyou.model;
	
	import com.zhqx.missyou.util.MapAndJson;
	import lombok.*;
	import org.hibernate.annotations.Where;
	
	import javax.persistence.*;
	import java.util.Map;
	
	import static javax.persistence.GenerationType.IDENTITY;
	
	@NoArgsConstructor
	@AllArgsConstructor
	@Entity
	@Getter
	@Setter
	@Where(clause = "delete_time is null")
	public class User extends BaseEntity {
	
	    @Id
	    @GeneratedValue(strategy = IDENTITY)
	    private Long id;
	    private String openid;
	
	    private String nickname;
	
	    private String email;
	
	    private String mobile;
	
	    private String password;
	
	    private Long unifyUid;
	
	    @Convert(converter = MapAndJson.class)
	    private Map<String, Object> wxProfile;
	
	}

UserRepository类:

	package com.zhqx.missyou.repository;
	
	import com.zhqx.missyou.model.User;
	import org.springframework.data.jpa.repository.JpaRepository;
	
	import java.util.Optional;
	
	public interface UserRepository extends JpaRepository<User, Long> {
	    User findByEmail(String email);
	    Optional<User> findByOpenid(String openid);
	    User findFirstById(Long id);
	    User findByUnifyUid(Long uuid);
	}

添加jwt库:

        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.8.1</version>
        </dependency>

在配置文件resources/config/application-dev.yml中配置jwt相关配置项:

	server:
	  port: 8080
	spring:
	  datasource:
	    url: jdbc:mysql://localhost:3306/sleeve?characterEncoding=utf-8&serverTimezone=GMT%2B8
	    username: root
	    password: root
	  jpa:
	    show-sql: true
	    properties:
	      hibernate:
	        format_sql: true
	
	zhqx:
	  security:
	    jwt-key: marshu
	    token-expired-in: 86400000



JwtToken类:

	package com.zhqx.missyou.util;
	
	import com.auth0.jwt.JWT;
	import com.auth0.jwt.JWTVerifier;
	import com.auth0.jwt.algorithms.Algorithm;
	import com.auth0.jwt.exceptions.JWTVerificationException;
	import com.auth0.jwt.interfaces.Claim;
	import com.auth0.jwt.interfaces.DecodedJWT;
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.stereotype.Component;
	
	import java.util.*;
	
	@Component
	public class JwtToken {
	
	    private static String jwtKey;
	    private static Integer expiredTimeIn;
	    //默认权限级别
	    private static Integer defaultScope = 8;
	
	    @Value("${zhqx.security.jwt-key}")
	    public void setJwtKey(String jwtKey) {
	        JwtToken.jwtKey = jwtKey;
	    }
	
	    @Value("${zhqx.security.token-expired-in}")
	    public void setExpiredTimeIn(Integer expiredTimeIn) {
	        JwtToken.expiredTimeIn = expiredTimeIn;
	    }
	
	    public static Optional<Map<String, Claim>> getClaims(String token) {
	        DecodedJWT decodedJWT;
	        Algorithm algorithm = Algorithm.HMAC256(JwtToken.jwtKey);
	        JWTVerifier jwtVerifier = JWT.require(algorithm).build();
	        try {
	            decodedJWT = jwtVerifier.verify(token);
	        } catch (JWTVerificationException e) {
	            return Optional.empty();
	        }
	        return Optional.of(decodedJWT.getClaims());
	    }
	
	    public static Boolean verifyToken(String token) {
	        try {
	            Algorithm algorithm = Algorithm.HMAC256(JwtToken.jwtKey);
	            JWTVerifier verifier = JWT.require(algorithm).build();
	            verifier.verify(token);
	        } catch (JWTVerificationException e) {
	            return false;
	        }
	        return true;
	    }
	
	
	    public static String makeToken(Long uid, Integer scope) {
	        return JwtToken.getToken(uid, scope);
	    }
	
	    public static String makeToken(Long uid) {
	        return JwtToken.getToken(uid, JwtToken.defaultScope);
	    }
	
	    private static String getToken(Long uid, Integer scope) {
	        Algorithm algorithm = Algorithm.HMAC256(JwtToken.jwtKey);
	        //计算过期时间
	        Map<String,Date> map = JwtToken.calculateExpiredIssues();
	        return JWT.create()
	                .withClaim("uid", uid)
	                .withClaim("scope", scope)
	                .withExpiresAt(map.get("expiredTime"))//过期时间
	                .withIssuedAt(map.get("now"))//签发时间
	                .sign(algorithm);
	    }
	
	    //当前时间+过期时间 就是令牌最终失效时间
	    private static Map<String, Date> calculateExpiredIssues() {
	        Map<String, Date> map = new HashMap<>();
	        Calendar calendar = Calendar.getInstance();
	        Date now = calendar.getTime();
	        calendar.add(Calendar.SECOND, JwtToken.expiredTimeIn);
	        map.put("now", now);
	        map.put("expiredTime", calendar.getTime());
	        return map;
	    }
	}

定义ParameterException:
	
	package com.zhqx.missyou.exception.http;
	
	public class ParameterException extends HttpException {
	    public ParameterException(int code){
	        this.code = code;
	        this.httpStatusCode = 400;
	    }
	}

在resources/config/exception-code.properties中配置ParameterException异常信息:

	zhqx.codes[9999] = 服务器未知异常O(∩_∩)O哈哈~
	
	zhqx.codes[10000] = 通用异常
	zhqx.codes[10001] = 通用参数异常
	zhqx.codes[10003] = 没有找到合适的登陆处理方法
	
	zhqx.codes[20004] = 获取用户wx openid失败
	
	zhqx.codes[30003] = 商品信息不存在
	zhqx.codes[30006] = Spu类的资源不存在

修改WxAuthenticationServiceImpl类:

	package com.zhqx.missyou.service.impl;
	
	import com.fasterxml.jackson.core.JsonProcessingException;
	import com.fasterxml.jackson.databind.ObjectMapper;
	import com.zhqx.missyou.exception.http.ParameterException;
	import com.zhqx.missyou.model.User;
	import com.zhqx.missyou.repository.UserRepository;
	import com.zhqx.missyou.service.WxAuthenticationService;
	import com.zhqx.missyou.util.JwtToken;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.stereotype.Service;
	import org.springframework.web.client.RestTemplate;
	
	import java.text.MessageFormat;
	import java.util.HashMap;
	import java.util.Map;
	import java.util.Optional;
	
	@Service
	public class WxAuthenticationServiceImpl implements WxAuthenticationService {
	
	    @Autowired
	    private ObjectMapper mapper;
	
	    @Autowired
	    private UserRepository userRepository;
	
	    @Value("${wx.code2session}")
	    private String code2SessionUrl;
	    @Value("${wx.appid}")
	    private String appid;
	    @Value("${wx.appsecret}")
	    private String appsecret;
	
	    public String code2Session(String code) {
	        String url = MessageFormat.format(this.code2SessionUrl, this.appid, this.appsecret, code);
	        RestTemplate rest = new RestTemplate();
	        Map<String, Object> session = new HashMap<>();
	        String sessionText = rest.getForObject(url, String.class);
	        try {
	            session = mapper.readValue(sessionText, Map.class);
	        } catch (JsonProcessingException e) {
	            e.printStackTrace();
	        }
	        return this.registerUser(session);
	    }
	
	    private String registerUser(Map<String, Object> session) {
	        String openid = (String)session.get("openid");
	        if (openid == null){
	            throw new ParameterException(20004);
	        }
	        Optional<User> userOptional = this.userRepository.findByOpenid(openid);
	        if(userOptional.isPresent()){//用户存在时
	            //TODO 数字等级
	            return JwtToken.makeToken(userOptional.get().getId());
	        }
	        User user = new User();
	        user.setOpenid(openid);
	        userRepository.save(user);
	        Long uid = user.getId();
	        return JwtToken.makeToken(uid);
	    }
	}

### jwt-token验证  ###
因为在之前的JWT中,已经预先引入了scope的概念(访问权限),自定义权限注解@ScopeLevel:

	package com.zhqx.missyou.core.interceptors;
	
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;
	
	@Target({ElementType.METHOD, ElementType.TYPE})
	@Retention(RetentionPolicy.RUNTIME)
	public @interface ScopeLevel {
	    int value() default 4;
	}


定义拦截器,拦截当前访问API用户token以及scope.

	package com.zhqx.missyou.core.interceptors;
	
	import com.auth0.jwt.interfaces.Claim;
	import com.zhqx.missyou.exception.http.ForbiddenException;
	import com.zhqx.missyou.exception.http.UnAuthenticatedException;
	import com.zhqx.missyou.model.User;
	import com.zhqx.missyou.service.UserService;
	import com.zhqx.missyou.util.JwtToken;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.util.StringUtils;
	import org.springframework.web.method.HandlerMethod;
	import org.springframework.web.servlet.ModelAndView;
	import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
	
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import java.util.Map;
	import java.util.Optional;
	
	public class PermissionInterceptor extends HandlerInterceptorAdapter {
	
	    public PermissionInterceptor() {
	        super();
	    }
	
	    @Override
	    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
	        Optional<ScopeLevel> scopeLevel = this.getScopeLevel(handler);
	        if (!scopeLevel.isPresent()) {//当前API访问没有权限注解,属于公开API
	            return true;
	        }
	        //获取请求头中携带的token信息 --规范上 前端会为token进行bearer加密
	        String bearerToken = request.getHeader("Authorization");
	        if (StringUtils.isEmpty(bearerToken)) {
	            throw new UnAuthenticatedException(10004);
	        }
	        if (!bearerToken.startsWith("Bearer")) {//不是标准的token
	            throw new UnAuthenticatedException(10004);
	        }
	        String tokens[] = bearerToken.split(" ");
	        if (!(tokens.length == 2)) {//为了避免有Bearer,但是没有token值的情况,对tokens长度进行判断
	            throw new UnAuthenticatedException(10004);
	        }
	        String token = tokens[1];
	        Optional<Map<String, Claim>> optionalMap = JwtToken.getClaims(token);
	        Map<String, Claim> map = optionalMap
	                .orElseThrow(() -> new UnAuthenticatedException(10004));
	        //比较当前token信息中的scopeLevel与接口中的scopeLevel
	        boolean valid = this.hasPermission(scopeLevel.get(), map);
	        return valid;
	    }
	
	    private boolean hasPermission(ScopeLevel scopeLevel, Map<String, Claim> map) {
	        Integer level = scopeLevel.value();
	        Integer scope = map.get("scope").asInt();
	        if (level > scope) {
	            throw new ForbiddenException(10005);
	        }
	        return true;
	    }
	
	    @Override
	    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
	        super.postHandle(request, response, handler, modelAndView);
	    }
	
	    @Override
	    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
	        super.afterCompletion(request, response, handler, ex);
	    }
	
	    //获取访问方法的@ScopeLevel注解
	    private Optional<ScopeLevel> getScopeLevel(Object handler) {
	        if (handler instanceof HandlerMethod) {
	            HandlerMethod handlerMethod = (HandlerMethod) handler;
	            ScopeLevel scopeLevel = handlerMethod.getMethod().getAnnotation(ScopeLevel.class);
	            if (scopeLevel == null) {
	                return Optional.empty();
	            }
	            return Optional.of(scopeLevel);
	        }
	        return Optional.empty();
	    }
	
	}

定义InterceptorConfiguration,用来注册拦截器

	package com.zhqx.missyou.core.configuration;
	
	import com.zhqx.missyou.core.interceptors.PermissionInterceptor;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.web.servlet.HandlerInterceptor;
	import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
	import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
	
	@Configuration
	public class InterceptorConfiguration implements WebMvcConfigurer {
	
	    @Bean
	    public HandlerInterceptor getPermissionInterceptor() {
	        return new PermissionInterceptor();
	    }
	
	    @Override
	    public void addInterceptors(InterceptorRegistry registry) {
	        registry.addInterceptor(this.getPermissionInterceptor());
	    }
	}

增加token验证API

	package com.zhqx.missyou.api.v1;
	
	import com.zhqx.missyou.dto.TokenDTO;
	import com.zhqx.missyou.dto.TokenGetDTO;
	import com.zhqx.missyou.exception.http.NotFoundException;
	import com.zhqx.missyou.service.WxAuthenticationService;
	import com.zhqx.missyou.util.JwtToken;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.validation.annotation.Validated;
	import org.springframework.web.bind.annotation.PostMapping;
	import org.springframework.web.bind.annotation.RequestBody;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	import java.util.HashMap;
	import java.util.Map;
	
	@RequestMapping(value = "token")
	@RestController
	public class TokenController {
	
	    @Autowired
	    private WxAuthenticationService wxAuthenticationService;
	
	    @PostMapping("")
	    public Map<String, String> getToken(@RequestBody @Validated TokenGetDTO userData) {
	        Map<String, String> map = new HashMap<>();
	        String token = null;
	        switch (userData.getType()) {
	            case USER_WX:
	                token = wxAuthenticationService.code2Session(userData.getAccount());
	                break;
	            case USER_EMAIL:
	                break;
	            default:
	                throw new NotFoundException(10003);
	        }
	        map.put("token", token);
	        return map;
	    }
	
	    @PostMapping("/verify")
	    public Map<String, Boolean> verify(@RequestBody TokenDTO token) {
	        Map<String, Boolean> map = new HashMap<>();
	        Boolean valid = JwtToken.verifyToken(token.getToken());
	        map.put("is_valid", valid);
	        return map;
	    }
	}

UserServiceImpl类:

	package com.zhqx.missyou.service.impl;
	
	import com.zhqx.missyou.model.User;
	import com.zhqx.missyou.repository.UserRepository;
	import com.zhqx.missyou.service.UserService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Service;
	
	import java.util.Map;
	
	@Service
	public class UserServiceImpl implements UserService {
	
	    @Autowired
	    private UserRepository userRepository;
	
	    @Override
	    public User getUserById(Long id) {
	        return userRepository.findFirstById(id);
	    }
	
	    @Override
	    public User createDevUser(Long uid) {
	        User newUser = new User();
	        newUser.setUnifyUid(uid);
	        userRepository.save(newUser);
	        return newUser;
	    }
	
	    @Override
	    public User getUserByUnifyUid(Long uuid) {
	        return userRepository.findByUnifyUid(uuid);
	    }
	}

前端请求需要携带令牌:

前端应该对获取的token进行处理:

	  _getBearerToken(){
	    const token = wx.getStorageSync('token')
	    return 'Bearer ' + token
	  }

前端请求时需要携带token:

	  onTestInterceptor(){
	    wx.request({
	      url: 'http://localhost:8080/v1/banner/name/b-1',
	      method: 'GET',
	      header:{
	        'Authorization': this._getBearerToken()
	      },
	      success: res => {
	        console.log(res.data)
	      }
	    })
	  },