---
layout: post
title: springboot-redis、rockectMQ
date: 2021-02-23 13:10:22
categories: springboot
tags: springboot
author: MarsHu
---

* content
{:toc}

### 订单的有效期,库存的归还  ###
预扣库存情况下,怎么归还库存,什么时候归还,谁来触发归还库存的操作。

什么时候:订单过期时,归还库存。

谁来触发:

主动轮询,不够精确。定时器,检查数据库所有到期订单,并且归还.

延迟消息队列:Redis\RocketMQ

Redis:键名通知---不能保证100%








### Redis键空间通知（KeySpaceNotifyfication）  ###
发布/订阅,假如我们存储了一个值: set key, name:xxx.当我们删除这个key时,就会触发事件通知。
key-space:del;key-event:name,具体的我们可以看redis的配置文件。

	#  K     Keyspace events, published with __keyspace@<db>__ prefix.
	#  E     Keyevent events, published with __keyevent@<db>__ prefix.
	#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
	#  $     String commands
	#  l     List commands
	#  s     Set commands
	#  h     Hash commands
	#  z     Sorted set commands
	#  x     Expired events (events generated every time a key expires)
	#  e     Evicted events (events generated when a key is evicted for maxmemory)
	#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.


开启:key-event通知订阅机制.在redis.conf中。找到EVENT NOTIFICATION配置项。按照上面的配置项,如果你想开启哪个通知,就配置哪个通知(注意不要加空格!!!,直接修改原来默认的哪一行,把""改成Ex即可)。

	notify-keyspace-events Ex

修改完配置文件后,默认打开redis是不加载配置文件的,所以需要手动指定配置文件,windows系统下,在reids目录下,启动PowerShell窗口,在窗口中输入如下命令:
	
	D:\server\Redis-x64-5.0.9\redis-server redis.windows.conf


可以使用`select 0`来切换到默认数据库(总共16个数据库,从0开始),先使用redis-cli来监听redis通知,输入如下命令开启订阅:

	psubscribe __keyevent@0__:expired

验证key过期事件,再次打开一个redis-cli,输入如下命令,设置一个key是name,过期时间是10秒,value值是hc的数据:

	setex name 10 hc 

	get name

返回之前输入订阅命令的redis-cli窗口,等待10秒,发现能够监听到过期,并且将过期的key名称返回。

	1) "pmessage"
	2) "__keyevent@0__:expired"
	3) "__keyevent@0__:expired"
	4) "name"

### SpringBoot中的Redis配置  ###
在application-dev.yml中增加redis配置,7号数据库,并监听key值过期:

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
	  redis:
	    localhost: localhost
	    port: 6379
	    database: 7
	    password:
	    listen-pattern: __keyevent@7__:expired

组合订单id号,用户id号,用来当作后面待保存到redis中的key值.安装redis库

	<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
       	</dependency>

存储到redis

	String key = uid.toString() + "," + oid.toString();
	stringRedisTemplate.opsForValue().set(key, "1", this.payTimeLimit, TimeUnit.SECONDS);


增加监听器:

	package com.zhqx.missyou.manager.redis;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.redis.connection.Message;
	import org.springframework.data.redis.connection.MessageListener;
	import org.springframework.stereotype.Component;
	
	@Component
	public class TopicMessageListener implements MessageListener {
	
	    @Override
	    public void onMessage(Message message, byte[] bytes) {
	        byte[] body = message.getBody();
	        byte[] channel = message.getChannel();
	
	        String expiredKey = new String(body);
	        String topic = new String(channel);
	
	        //处理逻辑---归还库存之类
	
	    }
	}

配置监听器:

	package com.zhqx.missyou.manager.redis;
	
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.data.redis.connection.RedisConnectionFactory;
	import org.springframework.data.redis.listener.PatternTopic;
	import org.springframework.data.redis.listener.RedisMessageListenerContainer;
	import org.springframework.data.redis.listener.Topic;
	
	@Configuration
	public class MessageListenerConfiguration {
	
	    @Value("${spring.redis.listen-pattern}")
	    public String pattern;
	
	    @Bean
	    public RedisMessageListenerContainer listenerContainer(RedisConnectionFactory redisConnection) {
	        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
	        container.setConnectionFactory(redisConnection);
	        //配置监听的事件是什么
	        Topic topic = new PatternTopic(this.pattern);
	        //添加自定义的监听器
	        container.addMessageListener(new TopicMessageListener(), topic);
	        return container;
	    }
	}



**在实际开发过程中,可以考虑在MessageListener中再次发布事件,具体的业务逻辑方法则有业务service监听执行。**


### 延迟消息队列  ###
RocketMQ本身是自带延迟消息队列的,而rabbitMQ和Kafka是需要通过一定插件去实现的。

订单数据库表:

	CREATE TABLE `order` (
	  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
	  `order_no` varchar(20) DEFAULT NULL COMMENT '订单号',
	  `user_id` int(10) unsigned DEFAULT NULL COMMENT 'user表外键',
	  `total_price` decimal(10,2) DEFAULT '0.00' COMMENT '订单总金额',
	  `total_count` int(11) unsigned DEFAULT '0' COMMENT '商品总数量',
	  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),
	  `delete_time` datetime(3) DEFAULT NULL,
	  `expired_time` datetime(3) DEFAULT NULL COMMENT '过期时刻',
	  `placed_time` datetime(3) DEFAULT NULL COMMENT '下单时间',
	  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),
	  `snap_img` varchar(255) DEFAULT NULL  COMMENT '订单封面',
	  `snap_title` varchar(255) DEFAULT NULL COMMENT '订单标题',
	  `snap_items` json DEFAULT NULL COMMENT '订单商品ids',
	  `snap_address` json DEFAULT NULL COMMENT '订单收获地址',
	  `prepay_id` varchar(255) DEFAULT NULL COMMENT '微信支付订单号',
	  `final_total_price` decimal(10,2) DEFAULT NULL COMMENT '最终价格(是否有优惠券)',
	  `status` tinyint(3) unsigned DEFAULT '1' COMMENT '订单状态',
	  PRIMARY KEY (`id`) USING BTREE,
	  UNIQUE KEY `uni_order_no` (`order_no`) USING BTREE
	) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4;

订单状态:

	package com.zhqx.missyou.core.enumeration;
	
	import java.util.stream.Stream;
	
	public enum OrderStatus {
	    All(0, "全部"),
	    UNPAID(1, "待支付"),
	    PAID(2, "已支付"),
	    DELIVERED(3, "已发货"),
	    FINISHED(4, "已完成"),
	    CANCELED(5, "已取消"),
	
	    //预扣除库存不存在以下这两种情况---预留状态,本项目中只有前面5种状态
	    PAID_BUT_OUT_OF(21, "已支付，但无货或库存不足"),
	    DEAL_OUT_OF(22, "已处理缺货但支付的情况");
	
	    private int value;
	
	    OrderStatus(int value, String text) {
	        this.value = value;
	    }
	
	    public int value() {
	        return this.value;
	    }
	
	    public static OrderStatus toType(int value) {
	        return Stream.of(OrderStatus.values())
	                .filter(c -> c.value == value)
	                .findAny()
	                .orElse(null);
	    }
	}

> **订单过期状态的3种解决方案总结:**

1.create_time + expired_time = expired_time_point  与 now比对

2.expired_time_point创建表时,就将该字段值记录到数据库种

3.使用reids / RocketMQ 判断状态  也不能保证100%,相对前面2种,更加精确

> **消息队列的应用场景:**

在传统应用中,假设A,B两个应用,A负责生产订单,B负责销售订单。传统模式中,A生产完,那么会入库,然后B再查询一次数据库.

通过消息队列,则可以将A生产的订单放入,由消息队列控制向B传输,这样就完成了A和B的解耦。

下载RocketMQ,`https://rocketmq.apache.org/docs/quick-start/`,安装官方说明,配置运行环境(这里需要注意的是,示例中配置的环境变量值是有""的,配置不能带""),windows下启动。

	错误: 找不到或无法加载主类 Files\Java\jdk1.8.0_192\jre\lib\ext

出现这个错误,是因为默认jdk安装目录带空格,改成没有空格的目录即可。改好后,启动Start Name Server

	D:\server\rocketmq-all-4.7.0-bin-release\bin\mqnamesrv.cmd

启动Start Broker

	D:\server\rocketmq-all-4.7.0-bin-release\bin\mqbroker.cmd -n localhost:9876 autoCreateTopicEnable=true

测试Send Messages:

	D:\server\rocketmq-all-4.7.0-bin-release\bin\tools.cmd  org.apache.rocketmq.example.quickstart.Producer

测试Receive Messages

	D:\server\rocketmq-all-4.7.0-bin-release\bin\tools.cmd  org.apache.rocketmq.example.quickstart.Consumer

在RocketMQ中内置了一组延迟时间级别:messageDelayLevel=1s 5s 10s 30m 1h...可以根据自己的需求,设置延迟消息队列。


### Springboot-RocketMQ延迟消息队列  ###
maven配置RocketMQ

	<dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>4.7.0</version>
        </dependency>

在配置文件中配置

	rocketmq:
	  consumer:
	    consumer-group: ZhqxConsumerGroup
	  producer:
	    producer-group: ZhqxProducerGroup
	  namesrv-addr: 127.0.0.1:9876

生产者:ProducerSchedule

	package com.zhqx.missyou.manager.rocketmq;
	
	import org.apache.rocketmq.client.exception.MQClientException;
	import org.apache.rocketmq.client.producer.DefaultMQProducer;
	import org.apache.rocketmq.client.producer.SendResult;
	import org.apache.rocketmq.common.message.Message;
	
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.stereotype.Component;
	
	import javax.annotation.PostConstruct;
	
	@Component
	public class ProducerSchedule {
	
	    private DefaultMQProducer producer;
	
	    @Value("${rocketmq.producer.producer-group}")
	    private String producerGroup;
	
	    @Value("${rocketmq.namesrv-addr}")
	    private String namesrvAddr;
	
	    public ProducerSchedule() {
	        //构造函数中无法取到producerGroup和namesrvAddr的值
	    }
	
	    @PostConstruct//初始化就执行的
	    public void defaultMQProducer() {
	        if (this.producer == null) {
	            this.producer = new DefaultMQProducer(this.producerGroup);
	            this.producer.setNamesrvAddr(this.namesrvAddr);
	        }
	        try {
	            this.producer.start();
	            System.out.println("-------producer start");
	        } catch (MQClientException e) {
	            e.printStackTrace();
	        }
	    }
	
	    //发送消息
	    public String send(String topic, String messageText) throws Exception {
	        Message message = new Message(topic, messageText.getBytes());
	        //TODO 可以写到配置文件,与订单过期时间一致
	        //messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
	        message.setDelayTimeLevel(4);//代表30s
	
	        SendResult result = this.producer.send(message);
	        System.out.println(result.getMsgId());
	        System.out.println(result.getSendStatus());
	        return result.getMsgId();
	    }
	
	
	}

消费者:ConsumerSchedule

	package com.zhqx.missyou.manager.rocketmq;
	
	import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
	import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
	import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
	import org.apache.rocketmq.client.exception.MQClientException;
	import org.apache.rocketmq.common.message.Message;
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.boot.CommandLineRunner;
	import org.springframework.stereotype.Component;
	
	@Component
	public class ConsumerSchedule implements CommandLineRunner {
	
	    @Value("${rocketmq.consumer.consumer-group}")
	    private String consumerGroup;
	
	    @Value("${rocketmq.namesrv-addr}")
	    private String namesrvAddr;
	
	    public void messageListener() throws MQClientException {
	        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer(consumerGroup);
	
	        consumer.setNamesrvAddr(namesrvAddr);
	
	        consumer.subscribe("TopicTest", "*");
	
	        consumer.setConsumeMessageBatchMaxSize(1);
	
	        consumer.registerMessageListener((MessageListenerConcurrently) (messages, context) -> {
	            for (Message message : messages) {
	                //TODO 取消订单
	                System.out.println("消息：" + new String(message.getBody()));
	            }
	            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
	        });
	
	        consumer.start();
	    }
	
	
	    @Override
	    public void run(String... args) throws Exception {
	        this.messageListener();
	    }
	}


测试控制器:

	package com.zhqx.missyou.api.v1;
	
	import com.zhqx.missyou.manager.rocketmq.ProducerSchedule;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	@RestController
	@RequestMapping("/test")
	public class TestController {
	
	    @Autowired
	    private ProducerSchedule producerSchedule;
	
	
	    @GetMapping("/push")
	    public void pushMessageToMQ() throws Exception {
	        producerSchedule.send("TopicTest", "test");
	    }
	}