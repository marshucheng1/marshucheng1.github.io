---
layout: post
title: springboot多线程提高服务性能
date: 2017-03-12 13:31:22
categories: JavaWeb
tags: springboot
author: MarsHu
---

* content
{:toc}

### springboot异步处理请求服务 ###
在实际开发中，有一些业务需求需要考虑高并发访问的情况，为了提高访问速度，就要求我们提高我们的请求服务性能。相关知识，学习自网上。

我们注意从以下三个方面学习异步处理请求服务：

1.使用`Runnable`异步处理请求服务

2.使用`DeferredResult`异步处理请求服务

3.异步处理的相关配置





### 为什么需要异步处理请求服务 ###
首先让我们看一张传统的同步处理的请求服务过程。

![xc1.png](http://marshucheng1.github.io/assets/thread/xc1.png)

由上图可以知道，传统的请求模式，虽然可以应对大多数应用场景。但是在应对高并发访问上，略有不足。

### 异步处理请求服务 ###
了解完传统的同步处理请求的过程，让我们再看看异步处理请求的过程。

![xc2.png](http://marshucheng1.github.io/assets/thread/xc2.png)

由上图可以知道，这种异步处理请求的模式，可以提高我们服务器的吞吐量

### 异步处理请求服务逻辑代码 ###
首先我们先看一下同步处理请求方式业务逻辑

	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	import lombok.extern.slf4j.Slf4j;
	
	@RestController
	@Slf4j
	public class AsyncController {
		
		@RequestMapping("/order")
		public String order() throws Exception {
			log.info("主线程开始");
			/*这里沉睡1秒,表示处理下单逻辑*/
			Thread.sleep(1000);
			log.info("主线程返回");
			return "success";
		}
	}

当我们访问下单服务时，我们可以从服务器控制台观察到如下日志记录
	
	INFO 14492 --- [nio-8080-exec-5] com.flarum.web.async.AsyncController     : 主线程开始
	INFO 14492 --- [nio-8080-exec-5] com.flarum.web.async.AsyncController     : 主线程返回

从日志结果来看，都是同一个线程在处理请求

下面，我们来看一下异步处理请求方式业务逻辑

	import java.util.concurrent.Callable;
	
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	import lombok.extern.slf4j.Slf4j;
	
	@RestController
	@Slf4j
	public class AsyncController {
		
		@RequestMapping("/order")
		public Callable<String> order() throws Exception {
			log.info("主线程开始");
			Callable<String> result = new Callable<String>() {
				@Override
				public String call() throws Exception {
					log.info("副线程开始");
					/*这里沉睡1秒,表示处理下单逻辑*/
					Thread.sleep(1000);
					log.info("副线程返回");
					return "success";
				}
			};
			log.info("主线程返回");
			return result;
		}
	}

其中的`Callable`表示在`spring`管理的线程中，再次单开一个线程来处理业务逻辑。在springboot2.0以上版本
执行运行并访问，会出现如下提示：

	An Executor is required to handle java.util.concurrent.Callable return values.
	Please, configure a TaskExecutor in the MVC config under "async support".
	The SimpleAsyncTaskExecutor currently in use is not suitable under load.

在`Spring 2.x`内部实现中，多处使用到`SimpleAsyncTaskExecutor`执行器。比如，JMS集成会使用

`SimpleAsyncTaskExecutor`完成`JMS`消息监听器的注册、执行等工作，我们将在第16章介绍这方面的内容。每次用户

提交新的任务给`SimpleAsyncTaskExecutor`时，它都会启动新的线程来响应客户请求，并在处理完客户请求后自动

销毁它。这意味着，它并没有提供线程池的功能。更何况，在`Java EE`环境中，随便启动新的线程并不是推荐的做

法，因为`Java EE`容器没有办法管理这些线程。所以，我们要寻求`SimpleAsyncTaskExecutor`的替代者。

怎么办~~~这个内容感觉对于当下的我感觉有点难以处理！百度。然后找到了一个不弹出提醒的处理方式。

参考文章地址：`http://book.51cto.com/art/201004/193424.htm`以及`https://blog.csdn.net/wuyufeng891021/article/details/86622670`

	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
	import org.springframework.web.context.request.async.TimeoutCallableProcessingInterceptor;
	import org.springframework.web.servlet.config.annotation.AsyncSupportConfigurer;
	import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
	
	@Configuration
	public class MyTaskExecutor implements WebMvcConfigurer {
		@Override
		public void configureAsyncSupport(final AsyncSupportConfigurer configurer) {
			configurer.setDefaultTimeout(60 * 1000L);
			configurer.registerCallableInterceptors(timeoutInterceptor());
			configurer.setTaskExecutor(threadPoolTaskExecutor());
		}
	
		@Bean
		public TimeoutCallableProcessingInterceptor timeoutInterceptor() {
			return new TimeoutCallableProcessingInterceptor();
		}
	
		@Bean
		public ThreadPoolTaskExecutor threadPoolTaskExecutor() {
			ThreadPoolTaskExecutor t = new ThreadPoolTaskExecutor();
			t.setCorePoolSize(10);
			t.setMaxPoolSize(100);
			t.setQueueCapacity(20);
			t.setThreadNamePrefix("WYF-Thread-");
			return t;
		}
	}

自定义自己的线程管理。注意这我们为后续创建的线程起了一个前缀名称：`WYF-Thread-`

这个时候我们通过浏览器访问一次请求，会在控制台看到如下结果：

	12:57:05.536  INFO 9452 --- [nio-8080-exec-1] com.flarum.web.async.AsyncController     : 主线程开始
	12:57:05.536  INFO 9452 --- [nio-8080-exec-1] com.flarum.web.async.AsyncController     : 主线程返回
	12:57:05.541  INFO 9452 --- [   WYF-Thread-1] com.flarum.web.async.AsyncController     : 副线程开始
	12:57:06.542  INFO 9452 --- [   WYF-Thread-1] com.flarum.web.async.AsyncController     : 副线程返回

我们可以看到主线程在一秒后就返回了，副线程在执行过程中时，主线程任然可以接收请求并处理。

### 更加复杂的应用场景DeferredResult ###
上面的多线程技术，虽然从某种程度上来说，极大的提高的服务器的处理性能。但是它并不适用更加复杂的应用场景。

让我们看一下一个更加复杂的应用场景：

![xc3.png](http://marshucheng1.github.io/assets/thread/xc3.png)

由于经验水平有限，只能够跟着后面学习。让我们看代码，理解下先：

模拟定义消息队列：

	import org.springframework.stereotype.Component;
	
	import lombok.extern.slf4j.Slf4j;
	/*代表消息队列*/
	@Component
	@Slf4j
	public class MockQueue {
	
		/*下单消息*/
		private String placeOrder;
		/*下单完成消息*/
		private String completeOrder;
	
		public String getPlaceOrder() {
			return placeOrder;
		}
	
		public void setPlaceOrder(String placeOrder) throws Exception {
			new Thread(() -> {
				log.info("接到下单请求, " + placeOrder);
				try {
					/*模拟下单过程*/
					Thread.sleep(1000);
				} catch (Exception e) {
					e.printStackTrace();
				}
				this.completeOrder = placeOrder;
				log.info("下单请求处理完毕," + placeOrder);
			}).start();
		}
	
		public String getCompleteOrder() {
			return completeOrder;
		}
	
		public void setCompleteOrder(String completeOrder) {
			this.completeOrder = completeOrder;
		}
	
	}

模拟定义订单处理服务器:

	import java.util.HashMap;
	import java.util.Map;
	
	import org.springframework.stereotype.Component;
	import org.springframework.web.context.request.async.DeferredResult;
	/*代表处理服务器*/
	@Component
	public class DeferredResultHolder {
		
		/*String:代表订单号;DeferredResult<String>:代表处理结果*/
		private Map<String, DeferredResult<String>> map = new HashMap<String, DeferredResult<String>>();
	
		public Map<String, DeferredResult<String>> getMap() {
			return map;
		}
	
		public void setMap(Map<String, DeferredResult<String>> map) {
			this.map = map;
		}
		
	}

模拟下单过程-线程：

	import org.apache.commons.lang.RandomStringUtils;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	import org.springframework.web.context.request.async.DeferredResult;
	
	import lombok.extern.slf4j.Slf4j;
	
	@RestController
	@Slf4j
	public class AsyncController {
		
		@Autowired
		private MockQueue mockQueue;
		
		@Autowired
		private DeferredResultHolder deferredResultHolder;
	
		@RequestMapping("/order")
		public DeferredResult<String> order() throws Exception {
			log.info("主线程开始");
			/*生成随机订单号,放入消息队列中*/
			String orderNumber = RandomStringUtils.randomNumeric(8);
			mockQueue.setPlaceOrder(orderNumber);
			/*处理订单*/
			DeferredResult<String> result = new DeferredResult<>();
			deferredResultHolder.getMap().put(orderNumber, result);
			log.info("主线程返回");
			return result;
		}
	}

模拟下单监听过程-线程：

	import org.apache.commons.lang.StringUtils;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.ApplicationListener;
	import org.springframework.context.event.ContextRefreshedEvent;
	import org.springframework.stereotype.Component;
	
	import lombok.extern.slf4j.Slf4j;
	
	//消息队列监听器
	//用来监听消息队列中,是否有订单处理完成的消息
	@Component
	@Slf4j
	public class QueueListener implements ApplicationListener<ContextRefreshedEvent> {
	
		@Autowired
		private MockQueue mockQueue;
		@Autowired
		private DeferredResultHolder deferredResultHolder;
	
		@Override
		public void onApplicationEvent(ContextRefreshedEvent event) {
			//将循环放入线程中,防止主线程阻死
			new Thread(() -> {
				while (true) {
					//当队列中表示订单完成的值不为空时,表示有订单处理完成,处理订单结果
					if (StringUtils.isNotBlank(mockQueue.getCompleteOrder())) {
						String orderNumber = mockQueue.getCompleteOrder();
						log.info("返回订单处理结果:"+orderNumber);
						deferredResultHolder.getMap().get(orderNumber).setResult("place order success");
						//订单处理完成后,更改订单处理状态
						mockQueue.setCompleteOrder(null);	
					}else{
						//如果没有监听到处理完毕的值,让线程沉睡1毫秒,再次执行监听功能
						try {
							Thread.sleep(100);
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				}
			}).start();
		}
	}

运行程序，在浏览器中发送下单请求，可以在控制台，看到如下结果：

	17:18:45.891  INFO 2668 --- [nio-8080-exec-1] com.flarum.web.async.AsyncController     : 主线程开始
	17:18:45.892  INFO 2668 --- [nio-8080-exec-1] com.flarum.web.async.AsyncController     : 主线程返回
	17:18:45.892  INFO 2668 --- [      Thread-11] com.flarum.web.async.MockQueue           : 接到下单请求, 05582014
	17:18:46.893  INFO 2668 --- [      Thread-11] com.flarum.web.async.MockQueue           : 下单请求处理完毕,05582014
	17:18:46.978  INFO 2668 --- [       Thread-8] com.flarum.web.async.QueueListener       : 返回订单处理结果:05582014

从结果我们可以看到，程序一共有3个线程参与了下单过程。

### sprigboot异步处理相关配置 ###
有关`springboot`异步处理的相关配置，实际上在使用`Callable`的时候，因为`springboot2.x`开始，默认的线程配置实际上是不可用的，建于我们手动配置。

这里的一些配置实际上就是解决那个警告信息的配置：

	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
	import org.springframework.web.context.request.async.TimeoutCallableProcessingInterceptor;
	import org.springframework.web.context.request.async.TimeoutDeferredResultProcessingInterceptor;
	import org.springframework.web.servlet.config.annotation.AsyncSupportConfigurer;
	import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
	
	@Configuration
	public class MyTaskExecutor implements WebMvcConfigurer {
		@Override
		public void configureAsyncSupport(final AsyncSupportConfigurer configurer) {
			configurer.setDefaultTimeout(60 * 1000L);
			/*Callable方式的--注册拦截器*/
			configurer.registerCallableInterceptors(callableTimeoutInterceptor());
			/*TimeoutDeferredResult方式的--注册拦截器*/
			configurer.registerDeferredResultInterceptors(timeoutDeferredTimeoutInterceptor());
			configurer.setTaskExecutor(threadPoolTaskExecutor());
		}
	
		@Bean
		public TimeoutCallableProcessingInterceptor callableTimeoutInterceptor() {
			return new TimeoutCallableProcessingInterceptor();
		}
		@Bean
		public TimeoutDeferredResultProcessingInterceptor timeoutDeferredTimeoutInterceptor() {
			return new TimeoutDeferredResultProcessingInterceptor();
		}
	
		@Bean
		public ThreadPoolTaskExecutor threadPoolTaskExecutor() {
			ThreadPoolTaskExecutor t = new ThreadPoolTaskExecutor();
			t.setCorePoolSize(10);
			t.setMaxPoolSize(100);
			t.setQueueCapacity(20);
			t.setThreadNamePrefix("WYF-Thread-");
			return t;
		}
	}

具体含义，后面有机会会再次解释，这里大家先自行看看把。这里值得注意的是，异步处理请求时，如果需要使用拦截器。和普通方式是有区别的。

`Callable`和`DeferredResult`方式如果需要使用拦截器，都需要使用不同的实现方式。