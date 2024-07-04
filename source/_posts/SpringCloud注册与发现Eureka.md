---
title: SpringCloud注册与发现Eureka
date: 2019-04-23 11:04:09
categories: 微服务
tags:
- SpringCloud
- Eureka
---

简介

Eureka是Netflix开发的服务发现框架，本身是一个基于REST的服务，主要用于定位运行在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。SpringCloud将它集成在其子项目spring-cloud-netflix中，以实现SpringCloud的服务发现功能。

Eureka包含两个组件：Eureka Server和Eureka Client。

Eureka Server  采用了 C-S 的设计架构,提供服务注册服务，各个节点启动后，会在Eureka Server中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到。

Eureka Client是一个java客户端，用于简化与Eureka Server的交互，客户端同时也就是一个内置的、使用轮询(round-robin)负载算法的负载均衡器。

在应用启动后，将会向Eureka Server发送心跳,默认周期为30秒，如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除(默认90秒)。

Eureka Server之间通过复制的方式完成数据的同步，Eureka还提供了客户端缓存机制，即使所有的Eureka Server都挂掉，客户端依然可以利用缓存中的信息消费其他服务的API。综上，Eureka通过心跳检查、客户端缓存等机制，确保了系统的高可用性、灵活性和可伸缩性。

Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务注册和发现(请对比Zookeeper)。

而系统中的其他微服务，使用 Eureka 的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。SpringCloud 的一些其他模块（比如Zuul）就可以通过 Eureka Server 来发现系统中的其他微服务，并执行相关的逻辑。

## 与Dubbo的架构对比

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423125949.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423130056.png)

## 小结

- Eureka Server 提供服务注册和发现
- Service Provider服务提供方将自身服务注册到Eureka，从而使服务消费方能够找到
- Service Consumer服务消费方从Eureka获取注册服务列表，从而能够消费服务

## microservicecloud-eureka-7001

### pom

pom文件配置信息。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.hph.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-eureka-7001</artifactId>

    <dependencies>
        <!--eureka-server服务端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
        <!-- 修改后立即生效，热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>

</project>
```

### application.yml

```yml
server: 
  port: 7001
 
eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    register-with-eureka: false #false表示不向注册中心注册自己。
    fetch-registry: false #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/        #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
```

### main

```java
package com.hph.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer //EurekaServer服务器端启动类,接受其它微服务注册进来

public class EurekaServer7001_App {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001_App.class, args);
    }
}
```

启动方法我们可以看到

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423113258.png)

看到上图说明Eureka的服务端就启动了，服务端启动之后我们将microservicecloud-provider-dept-8001的服务注册进服务端。

## microservicecloud-provider-dept-8001

### pom

继续上一章的内容，将服务注册进去我们需要添加一些依赖

```xml
   <!-- 将微服务provider侧注册进eureka -->
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-eureka</artifactId>
   </dependency>
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-config</artifactId>
   </dependency>
```

### main

```java
package com.hph.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient //服务启动后自动注册进入eureka服务
public class DeptProvider8001_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class,args);
    }
}
```

# 测试

启动DeptProvider8001_App，观察服务端信息。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423114444.png)

服务已经注册。

该微服务应用名称是由microservicecloud-provider-dept-8001中的application.yml指定的。 

```yaml
spring:
  application:
    name: microservicecloud-dept
```

## 问题1

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423120433.png)

当前的问题就是微服务的IP我们不知道，那么如何添加近服务的IP地址嗯我们可以在application中添加，

```yaml
  instance:
    instance-id: microservicecloud-dept8001   #自定义服务名称信息
    prefer-ip-address: true     #访问路径可以显示IP地址
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423120302.png)

## 问题2

我们点击超链接之后爆出了Error Page 如何解决这个问题呢？

首先我们修改 microservicecloud-provider-dept-8001中的pom文件

```xml
<dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
```

总的父工程microservicecloud修改pom.xml添加构建build信息。

```xml
   <build>
        <finalName>microservicecloud</finalName>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <configuration>
                    <delimiters>
                        <delimit>$</delimit>
                    </delimiters>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

在microservicecloud-provider-dept-8001中的application.yml中添加

```yaml
info:
  app.name: hph-microservicecloud
  company.name: www.hphblog.cn
  build.artifactId: ${project.artifactId}
  build.version: ${project.version}
```

热启动服务之后

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423125334.png)

# 自我保护

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423114022.png)

默认情况下，如果Eureka Server在一定时间内没有接收到某个微服务实例的心跳，Eureka Server将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务与Eureka Server之间无法正常通信，这就可能变得非常危险了----因为微服务本身是健康的，此时本不应该注销这个微服务。

Eureka Server通过“自我保护模式”来解决这个问题----当Eureka Server节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，Eureka Server就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该Eureka Server节点会自动退出自我保护模式。

 自我保护模式是一种对网络异常的安全保护措施。使用自我保护模式，而已让Eureka集群更加的健壮、稳定。

在分布式系统中有个著名的CAP定理（C-数据一致性；A-服务可用性；P-服务对网络分区故障的容错性，这三个特性在任何分布式系统中不能同时满足，最多同时满足两个）；

Netflix在设计Eureaka时遵循的时`AP原则`

## 复现

我们通过修改8001端口的服务名称然后再修改回来,使不同的服务注册到Eureka中。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423182510.png)

当我们点击进去的时候两个都可以点击进去提供的服务依旧可用，这是因为某时刻某一个微服务不可用了，eureka不会立刻清理，依旧会对该微服务的信息进行保存。

# 服务发现

首先我们再com.hph.springcloud.controller.DeptController中装配DiscoveryClient；

```java
@Autowired
private DiscoveryClient discoveryClient;


//添加服务
    @RequestMapping(value = "/deppt/discovery", method = RequestMethod.GET)
    public Object discovery() {
        List<String> list = discoveryClient.getServices();
        System.out.println("********" + list);
        List<ServiceInstance> serviceInstanceList = discoveryClient.getInstances("MICROSERVICECLOUD-DEPT");
        for (ServiceInstance serviceInstance : serviceInstanceList) {
            System.out.println(serviceInstance.getServiceId() + "\t" + serviceInstance.getHost() + "\t" + serviceInstance.getPort() + "\t" + serviceInstance.getUri());

        }

        return this.discoveryClient;

    }
```

再main方法中添加。

```java
@EnableEurekaClient //本服务启动后会自动注册进eureka服务中
@EnableDiscoveryClient //服务发现
```

启动EurekaServer7001_App服务端，和DeptProvider8001_App客户端。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423190331.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423190546.png)

在`microservicecloud-provider-dept-8001`中的`com.hph.springcloud.DeptProvider8001_Ap`中的main方法中添加注解

```java
@EnableEurekaClient //本服务启动后会自动注册进eureka服务中
@EnableDiscoveryClient //服务发现
```

消费端`microservicecloud-consumer-dept-80`中com.hph.springcloud.controller.DeptController_Consumer添加

```java
    // 测试@EnableDiscoveryClient,消费端可以调用服务发现
    @RequestMapping(value = "/consumer/dept/discovery")
    public Object discovery()
    {
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/discovery", Object.class);
    }
```

在`com.hph.springcloud.DeptConsumer80_App`中添加

```java
@EnableEurekaClient
//在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423201532.png)

# 集群配置

配置映射方便访问`C:\Windows\System32\drivers\etc\hosts`

```properties
127.0.0.1  eureka7001.com
127.0.0.1  eureka7002.com
127.0.0.1  eureka7003.com
```

新建`microservicecloud-eureka-7002`模块复制`icroservicecloud-eureka-7001`的启动类和配置文件进行相应的修改

```java
package com.hph.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer ////EurekaServer服务器端启动类,接受其它微服务注册进来

public class EurekaServer7002_App {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7002_App.class, args);
    }
}

```

```yaml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7003.com:7003/eureka/
```

```java
package com.hph.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer ////EurekaServer服务器端启动类,接受其它微服务注册进来

public class EurekaServer7003_App {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7003_App.class, args);
    }
}
```

```java
server:
  port: 7003

eureka:
  instance:
    hostname: eureka7003.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```

修改7001的文件为

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）。
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423204731.png)

集群配置成功，点击`microservicecloud-dept8001`查看是否跳转。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423205436.png)

可见对资源的消耗还是比较严重的，同时我们也差不多可以明白了为什么一个微服务就是一个进程这个说法。

# 与Zookeeper对比

**著名的CAP理论指出，一个分布式系统不可能同时满足C（一致性）、A（可用性）、和P（分区容错性）。由于分区容错性P在分布式系统中必须要保证的，因此我们只能在A和C之间进行权衡。**

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Eureka/20190423211438.png)

**因此：** 
**Zookeeper保证的是CP，** 
**Eureka则是AP。**

Zoopkeeper保证CP： 
当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但是不能接受服务直接down掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。但是zk会出现这样的一种情况，当master节点因网路故障与其他节点失去联系时，剩余的节点会重新进行leader选举。问题在于，选举leader的时间太长，30~120s，且选举期间整个zk集群是都是不可用的，这就导致在选举期间注册服务瘫痪，在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。

Eureka保证AP： 
Eureka看明白了这一点，因此在设计时就优先保证可用性。<font color="red">Eureka各个节点都是平等的，</font>几个节点挂掉不影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册时如果发现连接失败，则会自动切换至其他的节点，只要有一台Eureka还在，就能保证注册服务可用（保证可用性），只不过查到的信息可能不是最新的（不保证一致性）。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况： 
1.Eureka不再从注册列表中移除因为长时间没有收到心跳而应该过期的服务 
2.Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上（即保证当前节点依然可用） 
3.当前网络稳定时，当前实例新的注册信息会被同步到其它节点中 
<font color="red">因此，Eureka可以很好的应对因网络故障导致节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。</font>























