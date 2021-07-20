---
layout: post
title: Springboot基础知识点
date: 2021-02-15 12:10:22
categories: springboot
tags: springboot
author: MarsHu
---

* content
{:toc}

### Springboot版本介绍  ###

以`2.2.2.RELEASE`为例：

第一个2：主版本

第二个2：次版本 新特性 发布一些新特性 正常情况下要保证兼容

第三个2：增量版本 主要用于一些bug的修复 正常情况下要保证兼容

RELEASE：发布版本、里程碑版本
GA：正式版本 正常情况下，选择此版本
SNAPSHOT：快照版本

### stereotype annotations 模式注解  ###

@Component //@Conponent 是基础  

@Service
@Controller
@Repository

@Configuration







### 依赖注入时机与延迟实例化  ###

默认情况下，容器启动时，就会实例化。如果需要改变这个机制，可以在组件上使用@Lazy注解（在相关调用的组件上也需要是用@Lazy注解）

@Autowired(required = false) //允许注入对象是null（即对象没有使用@Component加载进容器）


### 属性注入、Setter注入、构造器注入  ###

	//假设Demo是个接口

	//属性注入--字段注入、成员变量注入
	public class TestController {
		//默认会有黄色提示,不推荐使用属性注入方式,推荐使用构造器注入
		@Autowired
		private  Demo demo;
	}

	//Setter注入
	public class TestController {
		
		private  Demo demo;

		//在属性的Setter方法上加上注解
		@Autowired
		public void setDemo(Demo demo) {
			this.demo = demo;
		}
	}

	//构造器注入
	public class TestController {
		
		private  Demo demo;

		//这里其实不添加@Autowired注解也是能够自动识别的,但是建议加上
		@Autowired
		public TestController(Demo demo) {
			this.demo = demo;
		}
	}

### @Autowired注入方式  ###

byName:按名称注入、byType:按类型注入（默认注入方式）

	public class TestController {
		//byType:自动寻找Demo接口实现类,然后进行初始化(要求Demo接口只有1个实现类,当Demo接口有多个实现类时,出错)
		//当不止一个实现时,不一定会报错,会先根据成员变量的名称,寻找对应名称的实现类进行初始化
		@Autowired
		private  Demo demo;
	}

	//当接口有多个实现时,可以使用@Qualifier注解
	public class TestController {
		//当使用@Qualifier注解时,就算成员变量的名称时demo2,实际上注入的仍然是demo1
		@Autowired
		@Qualifier("demo1")
		private  Demo demo2;
	}

如果需要注入一个静态的成员对象时,直接在成员变量上使用@Autowired是不正确的,应该在setter方法上使用:

	public class TestController {
		
		private static  Demo demo;

		//在属性的Setter方法上加上注解
		@Autowired
		public void setDemo(Demo demo) {
			TestController.demo = demo;
		}
	}



### @Configuration配置类  ###
@Configuration主要用来替代以前使用xml文件配置类的初始化。

当我们希望类可以被容器自动管理时,我们通常使用@Component注解。

	@Component
	public class Demo1 implements Demo {
		String name;
		Integer age;
		public Demo1(String name, Integer age) {
			this.name = name;
			this.age = age;
		}
	}

当不适用@Component注解时,我们可以通过使用@Configuration和@Bean注解来实现

	@Configuration
	public class Demo1Configuration {
		
		@Bean
		public Demo demo1() {
			return new Demo1("name", 17);
		}
	}

配合@Value注解,可以将配置文件中的值赋值给成员属性

假设applicationContext配置文件中有如下配置

	mysql.ip=127.0.0.1
	mysql.port=3306

使用@Value注解将值取出并赋值

	@Configuration
	public class DataBaseConfiguration {

		@Value("${mysql.ip}")
		private String ip;

		@Value("${mysql.port}")
		private Integer port;
		
		@Bean
		public Connect mysql() {
			return new Mysql(ip, port);
		}
	}

### @ComponentScan包扫描机制  ###
Springboot 默认情况下只会扫描到于启动类平级或者属于启动类平级目录下子目录中的Controller控制器。

如果控制器的目录级别高于启动类,尽管启动不报错,但是是没有办法访问到控制器的。

可以在启动类上使用@ComponentScan注解添加要访问的包

	
	//默认情况下只能扫描到missyou包下的
	//使用了注解后,可以扫描到zhqx报下的
	package com.zhqx.missyou;
	
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.context.annotation.ComponentScan;
	
	@SpringBootApplication
	@ComponentScan("com.zhqx")
	public class MissyouApplication {
	
	    public static void main(String[] args) {
	        SpringApplication.run(MissyouApplication.class, args);
	    }
	
	}

### 条件注解  ###
通过使用@Conditional注解加上Condition接口的实现类来控制是否初始化某个bean

	import org.springframework.context.annotation.Condition;
	import org.springframework.context.annotation.ConditionContext;
	import org.springframework.core.type.AnnotatedTypeMetadata;
	
	public class Demo1Condition implements Condition {
	
	    @Override
	    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
	        //通过相应的业务逻辑来控制返回true还是false
	        return true;
	    }
	}	

	import org.springframework.context.annotation.Condition;
	import org.springframework.context.annotation.ConditionContext;
	import org.springframework.core.type.AnnotatedTypeMetadata;
	
	public class Demo2Condition implements Condition {
	
	    @Override
	    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
	        //通过相应的业务逻辑来控制返回true还是false
	        return true;
	    }
	}	

	@Configuration
	public class DemoConfiguration {
	
	    @Bean
	    @Conditional(Demo1Condition.class)
	    public Demo demo1() {
	        return new Demo1();
	    }
	
	    @Bean
	    @Conditional(Demo2Condition.class)
	    public Demo demo2() {
	        return new Demo2();
	    }
	}


假设配置文件application.properties中有如下配置

	demo.condition=demo1

修改接口业务逻辑,通过读取配置文件来控制具体需要初始化哪个bean

	import org.springframework.context.annotation.Condition;
	import org.springframework.context.annotation.ConditionContext;
	import org.springframework.core.type.AnnotatedTypeMetadata;
	
	public class Demo1Condition implements Condition {
	
	    @Override
	    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
	        //通过相应的业务逻辑来控制返回true还是false
			//可以获取在配置文件中配置的值
	        String name = conditionContext.getEnvironment().getProperty("demo.condition");
        	return "demo1".equalsIgnoreCase(name);
	    }
	}	

	import org.springframework.context.annotation.Condition;
	import org.springframework.context.annotation.ConditionContext;
	import org.springframework.core.type.AnnotatedTypeMetadata;
	
	public class Demo2Condition implements Condition {
	
	    @Override
	    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
	        //通过相应的业务逻辑来控制返回true还是false
			//可以获取在配置文件中配置的值
	        String name = conditionContext.getEnvironment().getProperty("demo.condition");
        	return "demo2".equalsIgnoreCase(name);
	    }
	}

条件注解也可以使用在类上,通过调节判断是否需要将类加载到容器当中。

spring提供了一些已经开发好的条件注解,我们可以直接使用,并不一定需要自己去实现

	import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Conditional;
	import org.springframework.context.annotation.Configuration;
	
	@Configuration
	public class DemoConfiguration {
	
		//如果配置文件中配置了demo.condition且值是demo1时,会初始化Demo1
		//matchIfMissing表示,如果配置文件中不存在demo.condition配置项时,会默认加载demo1
	    @Bean
    	@ConditionalOnProperty(value = "demo.condition", havingValue = "demo1", matchIfMissing = true)
    	public Demo demo1() {
        	return new Demo1();
    	}

    	@Bean
    	@ConditionalOnProperty(value = "demo.condition", havingValue = "demo2")
    	public Demo demo2() {
        	return new Demo2();
    	}
	}

还有其他一些条件注解：

	@ConditionalOnBean("xxx")：当程序中存在某个名称的bean时。
	@ConditionalOnMissingBean("xxx")：当程序中不存在某个名称的bean时


### @SpringBootApplication注解的理解 ###

通过@EnableAutoConfiguration注解来实现自动加载

@Import({AutoConfigurationImportSelector.class})

public String[] selectImports(AnnotationMetadata annotationMetadata)

SpringFactoriesLoader

通过spring-boot-autoconfigure-x.x.x.jar包来管理需要自动装备的类,META-INF文件夹下有个spring.factories文件中配置了需要自动加载的类