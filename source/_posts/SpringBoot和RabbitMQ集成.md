---


title: SpringBoot和RabbitMQ集成
date: 2019-04-11 18:29:09
tags: RabbitMQ
categories: SpringBoot
---

步骤

![AH1g2V.png](https://s2.ax1x.com/2019/04/11/AH1g2V.png)

## 自动配置

```java
	@Bean
		public CachingConnectionFactory rabbitConnectionFactory(RabbitProperties config)
				throws Exception {
			RabbitConnectionFactoryBean factory = new RabbitConnectionFactoryBean();
			if (config.determineHost() != null) {
                //设置mq的host地址
				factory.setHost(config.determineHost());
			}
			factory.setPort(config.determinePort());
			if (config.determineUsername() != null) {
                //设置mq的username
				factory.setUsername(config.determineUsername());
			}
			if (config.determinePassword() != null) {
                //设置mq的密码
				factory.setPassword(config.determinePassword());
			}
			if (config.determineVirtualHost() != null) {
                //是指虚拟主机
				factory.setVirtualHost(config.determineVirtualHost());
			}
			if (config.getRequestedHeartbeat() != null) {
                //心跳
                factory.setRequestedHeartbeat(config.getRequestedHeartbeat());
            }
		.....
	}
```

```java
@ConfigurationProperties(prefix = "spring.rabbitmq")
public class RabbitProperties {
	//地址
   private String host = "localhost";
	//端口
   private int port = 5672;
	//账号
   private String username;
	//密码
   private String password;
	//SSL配置
   private final Ssl ssl = new Ssl();
    //虚拟主机
   private String virtualHost;
    //地址
   private String addresses;

	//请求心跳超时，以秒为单位; 零，没有。
   private Integer requestedHeartbeat;
    
	//Publisher Confirms and Returns机制
   private boolean publisherConfirms;
    
   private boolean publisherReturns;
	//连接超时时间
   private Integer connectionTimeout;
 	//缓存
   private final Cache cache = new Cache();

 	//监听容器配置
   private final Listener listener = new Listener();

   private final Template template = new Template();

   private List<Address> parsedAddresses;

   public String getHost() {
      return this.host;
   }
```

RabbitProperties封装了RabbitMQ发送和接收消息。

RabbitTemplate给RabbitMQ发送和接收消息。

AmqpAdmin,RabbitMQ系统管理功能组件。

```java
@Bean
 		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnMissingBean(RabbitTemplate.class)
		public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
            //生成rabbitTemplate来操作rabbitmq
			RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
			MessageConverter messageConverter = this.messageConverter.getIfUnique();
            //如果messageConverter不为空设置我们自己的messageConverter
			if (messageConverter != null) {
				rabbitTemplate.setMessageConverter(messageConverter);
			}
			rabbitTemplate.setMandatory(determineMandatoryFlag());
			RabbitProperties.Template templateProperties = this.properties.getTemplate();
			RabbitProperties.Retry retryProperties = templateProperties.getRetry();
			if (retryProperties.isEnabled()) {
				rabbitTemplate.setRetryTemplate(createRetryTemplate(retryProperties));
			}
			if (templateProperties.getReceiveTimeout() != null) {
				rabbitTemplate.setReceiveTimeout(templateProperties.getReceiveTimeout());
			}
			if (templateProperties.getReplyTimeout() != null) {
				rabbitTemplate.setReplyTimeout(templateProperties.getReplyTimeout());
			}
			return rabbitTemplate;
		}

		@Bean
		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnProperty(prefix = "spring.rabbitmq", name = "dynamic",
				matchIfMissing = true)
		@ConditionalOnMissingBean(AmqpAdmin.class)
		public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory) {
			return new RabbitAdmin(connectionFactory);
		}

	}
```

## P2P发送

```java
package com.hph.amqp;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootAmqpApplicationTests {


    @Autowired
    RabbitTemplate rabbitTemplate;

    /**
     * 单播 P2P
     */
    @Test
    public void p2p() {
        Map<String, Object> map = new HashMap<>();
        map.put("msg","这是第1个消息");
        map.put("data", Arrays.asList("Hello Rabitmq",123456, true));
		//对象默认被序列化以后发送出去
        rabbitTemplate.convertAndSend("exchange.direct", "phh.news",map);
    }

}
```

![AHdZeP.png](https://s2.ax1x.com/2019/04/11/AHdZeP.png)

这是因为默认使用的是application/x-java-serialized-object的序列化

## 获取消息

```java
    @Test
    public void receive() {
        Object o = rabbitTemplate.receiveAndConvert("hph.news");
        System.out.println(o.getClass());
        System.out.println(o);
    }
```

## 转为Json

由于是RabbitTemplate操作Rabbit的在RabbitTemplate中RabbitTemplate为默认的序列化器

```java
private volatile MessageConverter messageConverter = new SimpleMessageConverter();
```

MessageConverter又一下实现类我们使用的是Jackson2JsonMessageConverter的序列化器

![AHwo34.png](https://s2.ax1x.com/2019/04/11/AHwo34.png)

在设置我们自己的MessageConverter

```java
if (messageConverter != null) {
				rabbitTemplate.setMessageConverter(messageConverter);
			}
```

```java
package com.hph.amqp.config;

import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyAMQPConfig {
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

再次发送消息

![AHB9iT.png](https://s2.ax1x.com/2019/04/11/AHB9iT.png)

## 自定义发送

```java
  @Test
    public void sendMessage() {
        Map<String, Object> map = new HashMap<>();
        map.put("msg", "这是第1个消息");
        map.put("data", Arrays.asList("清风笑丶",123456,true));
        rabbitTemplate.convertAndSend("exchange.direct", "hph.news", new Person("小明",18));
    }

```

```java
package com.hph.amqp.bean;

public class Person {
    private String name;
    private Integer age;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

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

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

![AHBqk6.png](https://s2.ax1x.com/2019/04/11/AHBqk6.png)

## 反序列化

```java
    @Test
    public void receive() {
        Object o = rabbitTemplate.receiveAndConvert("hph.news");

        System.out.println(o.getClass());
        System.out.println(o);
    }
```

![AHcEOe.png](https://s2.ax1x.com/2019/04/11/AHcEOe.png)

## 广播发送

```java
    @Test
    public void sendMessages() {
        rabbitTemplate.convertAndSend("exchange.fanout", "hph.news", new Person("清风笑丶",18));
    }
```

![AHrePK.png](https://s2.ax1x.com/2019/04/11/AHrePK.png)

## 监听消息队列

```java
package com.hph.amqp;

import org.springframework.amqp.rabbit.annotation.EnableRabbit;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@EnableRabbit //开启基于注解的RabbitMQ的模式
@SpringBootApplication
public class SpringBootAmqpApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootAmqpApplication.class, args);
    }
}
```

```java
package com.hph.amqp.service;

import com.hph.amqp.bean.Person;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Service;

@Service
public class PersonService {

    @RabbitListener(queues = "hph.news")
    public void receive(Person person) {
        System.out.println("收到消息" + person+"上线");

    }
}
```

启动SpringBoot然后运行sendMessage任务。

![AHc1l8.png](https://s2.ax1x.com/2019/04/11/AHc1l8.png)

```java
   @RabbitListener(queues = "hph")
    public void receive02(Message message){
        System.out.println(message.getBody());
        System.out.println(message.getMessageProperties());
    }
}
```

![AHcWfx.png](https://s2.ax1x.com/2019/04/11/AHcWfx.png)

消息头信息。

## 管理

在SpringBoot中消息队列的管理使用到了amqpAdmin

```java
	@ConditionalOnMissingBean(AmqpAdmin.class)
		public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory) {
			return new RabbitAdmin(connectionFactory);
		}
```

在RabbitAutoConfiguration

![AHgiAs.png](https://s2.ax1x.com/2019/04/11/AHgiAs.png)

```java
public class DirectExchange extends AbstractExchange {

   public static final DirectExchange DEFAULT = new DirectExchange("");

	//设置名字
   public DirectExchange(String name) {
      super(name);
   }
	//名字  是否持久化 自动删除
   public DirectExchange(String name, boolean durable, boolean autoDelete) {
      super(name, durable, autoDelete);
   }

   public DirectExchange(String name, boolean durable, boolean autoDelete, Map<String, Object> arguments) {
      super(name, durable, autoDelete, arguments);
   }

   @Override
   public final String getType() {
      return ExchangeTypes.DIRECT;
   }

}
```

![

AHg0UA.png](https://s2.ax1x.com/2019/04/11/AHg0UA.png)

```java
   @Test
    public void createExchange(){
    amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange"));
        System.out.println("创建完成");
    }
```

运行该方法。

![AH2Yin.png](https://s2.ax1x.com/2019/04/11/AH2Yin.png)

### 创建exchange

```java
	public Queue(String name, boolean durable) {
		this(name, durable, false, false, null);
	}

	public Queue(String name, boolean durable, boolean exclusive, boolean autoDelete) {
		this(name, durable, exclusive, autoDelete, null);
	}

	public Queue(String name, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments) {
		Assert.notNull(name, "'name' cannot be null");
		this.name = name;
		this.durable = durable;
		this.exclusive = exclusive;
		this.autoDelete = autoDelete;
		this.arguments = arguments;
	}
```
### 创建Queue

```java
  @Test
    public void createQueue() {
        amqpAdmin.declareQueue(new Queue("amqpadmin.queue", true));
        System.out.println("创建队列成功");
    }
```

![AHRUkd.png](https://s2.ax1x.com/2019/04/11/AHRUkd.png)



### 绑定exchange

```java
public Binding(String destination, DestinationType destinationType, String exchange, String routingKey,
			Map<String, Object> arguments) {
		this.destination = destination;
		this.destinationType = destinationType;
		this.exchange = exchange;
		this.routingKey = routingKey;
		this.arguments = arguments;
	}
```

之前尚未绑定

![AHR5cV.png](https://s2.ax1x.com/2019/04/11/AHR5cV.png)

```java
    @Test
    public void bindExchange() {
        amqpAdmin.declareBinding(new Binding("amqpadmin.queue", Binding.DestinationType.QUEUE, "amqpadmin.exchange", "amqp.bind", null));
    }
```

![AHRqAJ.png](https://s2.ax1x.com/2019/04/11/AHRqAJ.png)

绑定成功





