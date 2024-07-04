---
title: SpringCloud的Ribbon负载均衡
date: 2019-04-25 18:26:50
categories: 微服务
tags:
- SpringCloud
- Ribbon
---

简介


Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。

简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法。

# 负载均衡

LB，即负载均衡(Load Balance)，在微服务或分布式集群中经常用的一种应用。负载均衡简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA。常见的负载均衡有软件Nginx，LVS，硬件 F5等。相应的在中间件，例如：dubbo和SpringCloud中均给我们提供了负载均衡，SpringCloud的负载均衡算法可以自定义。 

## 集中式LB

即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方；

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/20190425231310.png)

​														F5硬件图例

## 进程内LB

将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。

# 步骤

##  microservicecloud-consumer-dept-80

## pom.xml

添加

```xml
   <!-- Ribbon相关 -->
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-eureka</artifactId>
   </dependency>
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-ribbon</artifactId>
   </dependency>
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

## application.yml 

```yaml
server:
  port: 80
 
eureka:
  client:
    register-with-eureka: false
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

## ConfigBean

对ConfigBean进行新注解@LoadBalanced    获得Rest时加入Ribbon的配置

```java
package com.hph.springcloud.cfbeans;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ConfigBean {

    @Bean
    @LoadBalanced   //Spring Cloud Ribbon 是基于Netflix Ribbon实现的的一套客户端 负载 均衡的工具
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

## DeptConsumer80_App

 主启动类DeptConsumer80_App添加@EnableEurekaClient

```java
package com.hph.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
//在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效,如果没有启动是默认的轮询
public class DeptConsumer80_App
{
    public static void main(String[] args)
    {
        SpringApplication.run(DeptConsumer80_App.class, args);
    }
}

```

## DeptController_Consumer

修改DeptController_Consumer客户端访问类,把REST_URL_PREFIX写成我们的微服务名称。

```java
package com.hph.springcloud.controller;

import com.hph.springcloud.entities.Dept;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.List;

@RestController
public class DeptController_Consumer {

//    private  static  final  String REST_URL_PREFIX = "http://localhost:8001";
    private  static  final  String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";
    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping(value = "/consumer/dept/add")
    public boolean add(Dept dept)
    {
        return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
    }

    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id)
    {
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
    }

    @SuppressWarnings("unchecked")
    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list()
    {
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/list", List.class);
    }

    // 测试@EnableDiscoveryClient,消费端可以调用服务发现
    @RequestMapping(value = "/consumer/dept/discovery")
    public Object discovery()
    {
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/discovery", Object.class);
    }
}
```

## 测试

先启动3个eureka集群后，再启动`microservicecloud-provider-dept-8001`并注册进eureka

启动microservicecloud-consumer-dept-80

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/20190426110653.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/20190426110727.png)



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/20190426110813.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/20190426110934.png)

小结 `Ribbon和Eureka整合后Consumer可以直接调用服务而不用再关心地址和端口号`

# Ribbon架构

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/20190426110956.png)

Ribbon在工作时分成两步
第一步先选择 EurekaServer ,它优先选择在同一个区域内负载较少的server.
第二步再根据用户指定的策略，在从server取到的服务注册列表中选择一个地址。
其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。

# Ribbon负载均衡

## microservicecloud-provider-dept-8002

数据准备

```sql
 
DROP DATABASE IF EXISTS cloudDB02;
 
CREATE DATABASE cloudDB02 CHARACTER SET UTF8;
 
USE cloudDB02;
 
CREATE TABLE dept
(
  deptno BIGINT NOT NULL PRIMARY KEY AUTO_INCREMENT,
  dname VARCHAR(60),
  db_source   VARCHAR(60)
);
 
INSERT INTO dept(dname,db_source) VALUES('开发部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('人事部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('财务部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('市场部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('运维部',DATABASE());
 
SELECT * FROM dept;
```

### application.yml

```yaml
server:
  port: 8002

mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
  type-aliases-package: com.hph.springcloud.entities    # 所有Entity别名类所在包
  mapper-locations:
    - classpath:mybatis/mapper/**/*.xml                       # mapper映射文件

spring:
  application:
    name: microservicecloud-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://192.168.1.110:3306/cloudDB02         # 数据库名称
    username: root
    password: 123456
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200                                  # 等待连接获取的最大超时时间

eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      defaultZone:  http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: microservicecloud-dept8002   #自定义服务名称信息
    prefer-ip-address: true     #访问路径可以显示IP地址

info:
  app.name: hph-microservicecloud
  company.name: www.hphblog.cn
  build.artifactId: ${project.artifactId}
  build.version: ${project.version}

```

其他基本相同我们需要把启动类的配置类做适当修改,完整代码请参考Github链接。

## 测试

1. 启动3个eureka集群配置区

2. 启动3个Dept微服务并各自测试通过 

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/测试.gif)

3. 客户端通过Ribbo完成负载均衡并访问上一步的Dept微服务

    我们可以看得到的访问到的是不同的数据库信息。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/负载均衡动图.gif)

## 小结

Ribbon其实就是一个软负载均衡的客户端组件，他可以和其他所需请求的客户端结合使用，和eureka结合只是其中的一个实例。

# 自定义负载均衡

## Ribbon核心组件IRule

IRule：根据特定算法中从服务列表中选取一个要访问的服务

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/20190426122015.png)

## 指定轮询算法

### 创建com.hph.myrule

在指定中添加指定为随机轮询

```java
package com.hph.myrule;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;

public class MySelfRule {
    @Bean
    public IRule myRule() {
        return new RandomRule();//Ribbon默认是轮询，我自定义为随机

    }
}
```

修改我我们的主启动类

```java
package com.hph.springcloud;

import com.hph.myrule.MySelfRule;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.ribbon.RibbonClient;

@SpringBootApplication
@EnableEurekaClient
//在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效
@RibbonClient(name="MICROSERVICECLOUD-DEPT",configuration= MySelfRule.class)
public class DeptConsumer80_App
{
    public static void main(String[] args)
    {
        SpringApplication.run(DeptConsumer80_App.class, args);
    }
}
```



我们测试一下结果是随机访问你的微服务的数据库。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/随机算法测试.gif)

## 自定义注意

官方文档明确给出了警告：
这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下，否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，也就是说我们达不到特殊化定制的目的了。

因此我们创新创建一个包，不让自己定义的规则和SpirngApplication在同一包下，所以我们选择重新创建一个包

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/20190426135238.png)

这里我们可以看到相关的信息。根据[github源码](https://github.com/Netflix/ribbon/blob/master/ribbon-loadbalancer/src/main/java/com/netflix/loadbalancer/RandomRule.java)我们自己写一个轮询策略。



```java
package com.hph.myrule;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.Server;

import java.util.List;
 
public class RandomRule_ZY extends AbstractLoadBalancerRule {
 
  private int total = 0;    //总共被调用的次数，目前要求每台被调用5次
  private int currentIndex = 0;//当前提供服务的机器号
  
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;
 
        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            List<Server> upList = lb.getReachableServers();
            List<Server> allList = lb.getAllServers();
 
            int serverCount = allList.size();
            if (serverCount == 0) {
                /*
                 * No servers. End regardless of pass, because subsequent passes
                 * only get more restrictive.
                 */
                return null;
            }
            
//            int index = rand.nextInt(serverCount);
//            server = upList.get(index);
            if(total < 5)
            {
            server = upList.get(currentIndex);
            total++;
            }else {
            total = 0;
            currentIndex++;
            if(currentIndex >= upList.size())
            {
              currentIndex = 0;
            }
            
            }
           
       
            if (server == null) {
                /*
                 * The only time this should happen is if the server list were
                 * somehow trimmed. This is a transient condition. Retry after
                 * yielding.
                 */
                Thread.yield();
                continue;
            }
 
            if (server.isAlive()) {
                return (server);
            }
 
            // Shouldn't actually happen.. but must be transient or a bug.
            server = null;
            Thread.yield();
        }
 
        return server;
 
    }
 
  @Override
  public Server choose(Object key) {
   return choose(getLoadBalancer(), key);
  }
 
  @Override
  public void initWithNiwsConfig(IClientConfig clientConfig) {
   
  }
}
```

## 测试

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Ribbon/测试5次访问.gif)

自定义完成。

## 完整代码

Github地址 : https://github.com/bigdataxiaohan/microservicecloud/tree/master/Ribbon

















