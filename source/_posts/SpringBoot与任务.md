---
title: SpringBoot与任务
date: 2019-04-13 10:01:18
tags: 任务
categories: SpringBoot 
---

## 准备

![ALp4vq.png](https://s2.ax1x.com/2019/04/13/ALp4vq.png)

暂时只选中web模块

![ALpOPJ.png](https://s2.ax1x.com/2019/04/13/ALpOPJ.png)

## 异步任务

```java
package com.hph.task.service;

import org.springframework.stereotype.Service;

import java.text.SimpleDateFormat;
import java.util.Calendar;

@Service
public class Asyncservice {

    public void dataprocessing() {
        Calendar ago = Calendar.getInstance();
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd :hh:mm:ss");
        System.out.println("数据处理前"+dateFormat.format(ago.getTime()));

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("数据正在处理中......");

        Calendar now = Calendar.getInstance();
        System.out.println("数据处理完毕"+dateFormat.format(now.getTime()));
    }

}
```

```java
package com.hph.task.controller;

import com.hph.task.service.Asyncservice;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AsyncController {
    @Autowired
    Asyncservice asyncservice;

    @GetMapping("/dataprocessing")
    public String dataprocessing() {
        asyncservice.dataprocessing();
        return "success";
    }
}
```

三秒之后数据有响应。

![ALCytg.png](https://s2.ax1x.com/2019/04/13/ALCytg.png)

要完成数据的异步调用其实很简单我们只需要在SpringbootTaskApplication 开启异步注解功能

```java
package com.hph.task;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableAsync;

@EnableAsync //开启异步注解功能
@SpringBootApplication
public class SpringbootTaskApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootTaskApplication.class, args);
    }

}

```

```java
package com.hph.task.service;

import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.text.SimpleDateFormat;
import java.util.Calendar;

@Service
public class Asyncservice {
    @Async   //异步任务
    public void dataprocessing() {
        Calendar ago = Calendar.getInstance();
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd :hh:mm:ss");
        System.out.println("数据处理前" + dateFormat.format(ago.getTime()));

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("数据正在处理中......");

        Calendar now = Calendar.getInstance();
        System.out.println("数据处理完毕" + dateFormat.format(now.getTime()));
    }

}

```

![ALPnUS.png](https://s2.ax1x.com/2019/04/13/ALPnUS.png)

## 定时任务

定时任务可以按照自己的规则定时启动任务。

```java
public @interface Scheduled {
	//这个cron比较重要 比较像Linux中的crontab
 
	//         秒  分   时   日   月  周几
	// {@code "0   *    *    *    *  MON-FRI"} 周一到周五每秒启动一次
	String cron() default "";

	String zone() default "";

	long fixedDelay() default -1;

	String fixedDelayString() default "";

	long fixedRate() default -1;

	String fixedRateString() default "";

	long initialDelay() default -1;

	String initialDelayString() default "";

}
```

### 准备

#### service

```java
package com.hph.task.service;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

import java.text.SimpleDateFormat;
import java.util.Calendar;

@Service
public class ScheduledService {
    @Scheduled(cron = "0   *   *  *  *  MON-SAT") //每分钟启动一次周一到周六
    public void hello() {
        Calendar now = Calendar.getInstance();
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd :hh:mm:ss");
        System.out.println(dateFormat.format(now.getTime())+"  定时任务启动 ..  .. .. ");
    }
}
```

#### 开启注解

需要在`SpringbootTaskApplication`开启注解

```java
@SpringBootApplication
@EnableScheduling   //开启基于注解的定时任务
public class SpringbootTaskApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootTaskApplication.class, args);
    }
}
```

![ALiERJ.png](https://s2.ax1x.com/2019/04/13/ALiERJ.png)

#### cron表达式

![ALivFO.png](https://s2.ax1x.com/2019/04/13/ALivFO.png)

```java
@Scheduled(cron = "0,1,2,3,4   *   *  *  *  MON-SAT") //每分钟的头1-4秒启动定时任务
```

![ALFZtS.png](https://s2.ax1x.com/2019/04/13/ALFZtS.png)

```java
@Scheduled(cron = "0-4  *   *  *  *  MON-SAT") //每分钟的头1-4秒启动定时任务
```

![ALFD76.png](https://s2.ax1x.com/2019/04/13/ALFD76.png)

```java
@Scheduled(cron = "0/4  *   *  *  *  MON-SAT") //每4秒启动定时任务
```

![ALFRcd.png](https://s2.ax1x.com/2019/04/13/ALFRcd.png)

其他例子

```text
  	* 0 * * * * MON-FRI
     *  【0 0/5 14,18 * * ?】 每天14点整，和18点整，每隔5分钟执行一次
     *  【0 15 10 ? * 1-6】 每个月的周一至周六10:15分执行一次
     *  【0 0 2 ? * 6L】每个月的最后一个周六凌晨2点执行一次
     *  【0 0 2 LW * ?】每个月的最后一个工作日凌晨2点执行一次
     *  【0 0 2-4 ? * 1#1】每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次；
```

##  邮件任务

### 准备

我们需要在邮件中引入依赖

```xml
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
 </dependency>
```

### 自动配置

```java
@Configuration
@ConditionalOnClass(Session.class)
@ConditionalOnProperty(prefix = "spring.mail", name = "jndi-name")
@ConditionalOnJndi
class MailSenderJndiConfiguration {

	private final MailProperties properties;

	MailSenderJndiConfiguration(MailProperties properties) {
		this.properties = properties;
	}

	@Bean   //用来发送邮件的 
	public JavaMailSenderImpl mailSender(Session session) {
		JavaMailSenderImpl sender = new JavaMailSenderImpl();
		sender.setDefaultEncoding(this.properties.getDefaultEncoding().name());
		sender.setSession(session);
		return sender;
	}

	@Bean
	@ConditionalOnMissingBean
	public Session session() {
		String jndiName = this.properties.getJndiName();
		try {
			return new JndiLocatorDelegate().lookup(jndiName, Session.class);
		}
		catch (NamingException ex) {
			throw new IllegalStateException(
					String.format("Unable to find Session in JNDI location %s", jndiName),
					ex);
		}
	}

}

```

可以配置的选项

```java
@ConfigurationProperties(prefix = "spring.mail")
public class MailProperties {

	private static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

	private String host;
    
	private Integer port;

	private String username;

	private String password;

	private String protocol = "smtp";

	private Charset defaultEncoding = DEFAULT_CHARSET;

	private Map<String, String> properties = new HashMap<String, String>();

	private String jndiName;
    ......
}
```

### 配置邮箱

需要将QQ邮箱中设置一下

![ALV1v4.png](https://s2.ax1x.com/2019/04/13/ALV1v4.png)

在`application.properties`中配置

```properties
spring.mail.username=467008580@qq.com
spring.mail.password=meqkusfmrwxxbhag   #授权码
spring.mail.host=smtp.qq.com
```
### 简单邮件

```java
    @Autowired
    JavaMailSender mailSender;

    @Test
    public void sendMail() {
        SimpleMailMessage message = new SimpleMailMessage();
        //邮件设置
        message.setSubject("邮件测试通知来自QQ邮箱");
        message.setText("SpringBoot的邮件测试");
        message.setTo("han_penghui@sina.com"); //给新浪发送邮箱
        message.setFrom("467008580@qq.com");
        mailSender.send(message);
    }
```

启动测试类

![ALVQ8U.png](https://s2.ax1x.com/2019/04/13/ALVQ8U.png)

如果运行程序出错在`application.properties`中添加配置

```properties
spring.mail.properties.mail.smtp.ssl.enable=true
```

### 复杂邮件

```java
//复杂邮件发送需要将第二个参数设置为ture	
public MimeMessageHelper(MimeMessage mimeMessage, boolean multipart) throws MessagingException {
		this(mimeMessage, multipart, null);
}
```

```java
    @Test
    public void sendMimeMail() throws MessagingException {
        //创建一个复杂的消息右键
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,true);

        helper.setSubject("复杂邮件测试来自QQ邮箱");
        helper.setText("<b style='color:red'>SpringBoot</b>的<em>邮件测试</em>",true);  //如果没有设置true默认是false，标签不生效
        helper.setTo("han_penghui@sina.com"); //给新浪发送邮箱

        //helpr上传文件
        helper.addAttachment("背景.jpg",new File("E:\\mail\\bg.jpg"));
        helper.addAttachment("Java.pdf",new File("E:\\mail\\Java知识.pdf"));
        helper.setFrom("467008580@qq.com");
        mailSender.send(mimeMessage);

    }
```

![ALZ1FP.png](https://s2.ax1x.com/2019/04/13/ALZ1FP.png)



![ALZMdI.png](https://s2.ax1x.com/2019/04/13/ALZMdI.png)

发送成功



