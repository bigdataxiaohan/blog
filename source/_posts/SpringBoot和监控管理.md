---
title: SpringBoot和监控管理
date: 2019-04-14 21:29:47
tags: 
  - SpringBoot Admin
categories: SpringBoot
---

## 简介

引入spring-boot-starter-actuator，我们可以使用Spring Boot为我们提供的准生产环境下的应用监控和管理功能。我们可以通过HTTP，JMX，SSH协议来进行操作，自动得到审计、健康及指标信息。

## 准备

![AXV3rQ.png](https://s2.ax1x.com/2019/04/14/AXV3rQ.png)

我们在什么都不做的情况下启动项目

![AjILvD.png](https://s2.ax1x.com/2019/04/15/AjILvD.png)

这里映射了许多的方法比如把info映射到info.json调用的是哪一个方法。

![Ajo0xO.png](https://s2.ax1x.com/2019/04/15/Ajo0xO.png)

这个是我们引入了spring-boot-starter-actuator的原因如果我们不引入这个以来我们看一下效果会是怎么样的。

![AjTFF1.png](https://s2.ax1x.com/2019/04/15/AjTFF1.png)

心塞啊少了那么多信息，只剩下默认的/error了，可见这个模块的作用可以为我们提供的准生产环境下的应用监控和管理功能。下面我们访问一下它的关键的监控把。

## 关键功能

### beans

![AjTT1K.png](https://s2.ax1x.com/2019/04/15/AjTT1K.png)

需要我们在application.properties中添加

```yml
management.security.enabled=false
```

我们安装了热部署插件可以按住ctrl+F9来重新运行服务。

再次访问beans看到了不同的信息。beans是监控容器中的组件的信息的。

![Aj7ajO.png](https://s2.ax1x.com/2019/04/15/Aj7ajO.png)

### health

health的信息

![Aj7g8P.png](https://s2.ax1x.com/2019/04/15/Aj7g8P.png)

### info

info为空，那么info信息是怎么来的呢

![Aj7T5n.png](https://s2.ax1x.com/2019/04/15/Aj7T5n.png)

在配置文件中添加

```yml
info.app.id=app1
info.app.versio=1.0.0
```

热部署之后

![AjHSa9.png](https://s2.ax1x.com/2019/04/15/AjHSa9.png) 

### github

在开发过程中我们有时需要将源码托管给github，我们要配置一些github的属性。创建application.properties文件。

![AjHvSP.png](https://s2.ax1x.com/2019/04/15/AjHvSP.png)

### dump

暴露我们程序运行中的线程信息。

![Ajb0kd.png](https://s2.ax1x.com/2019/04/15/Ajb0kd.png)

### autoconfig

SpringBoot的自动配置信息。

![AjXXKx.png](https://s2.ax1x.com/2019/04/15/AjXXKx.png)

### heapdump

输入localhost:8080/heapdump即可下载信息

![AjzmdK.png](https://s2.ax1x.com/2019/04/15/AjzmdK.png)

### trace

![AjzaFS.png](https://s2.ax1x.com/2019/04/15/AjzaFS.png)



### mappings

![AjzdJg.png](https://s2.ax1x.com/2019/04/15/AjzdJg.png)

### metrics

![AjzoO1.png](https://s2.ax1x.com/2019/04/15/AjzoO1.png)

### env

![AvSZlj.png](https://s2.ax1x.com/2019/04/15/AvSZlj.png)

### configprops

![AvSK00.png](https://s2.ax1x.com/2019/04/15/AvSK00.png)

```properties
endpoints.metrics.enabled=false
```

![AvSd76.png](https://s2.ax1x.com/2019/04/15/AvSd76.png)

### shutdown

在配置文件中配置

```properties
endpoints.shutdown.enabled=true
```

![AvS4N8.png](https://s2.ax1x.com/2019/04/15/AvS4N8.png)

![AvS7cj.png](https://s2.ax1x.com/2019/04/15/AvS7cj.png)



### 总结

| **端点名**   | **描述**                    |
| ------------ | --------------------------- |
| *autoconfig* | 所有自动配置信息            |
| auditevents  | 审计事件                    |
| beans        | 所有Bean的信息              |
| configprops  | 所有配置属性                |
| dump         | 线程状态信息                |
| env          | 当前环境信息                |
| health       | 应用健康状况                |
| info         | 当前应用信息                |
| metrics      | 应用的各项指标              |
| mappings     | 应用@RequestMapping映射路径 |
| shutdown     | 关闭当前应用（默认关闭）    |
| trace        | 追踪信息（最新的http请求）  |

## 定制端点信息

```properties
endpoints.beans.id=myselfbean
endpoints.beans.path=/myselfbeanspath
```

![Avp1bt.png](https://s2.ax1x.com/2019/04/15/Avp1bt.png)

小结一下

```tex
定制端点一般通过endpoints+端点名+属性名来设置。
修改端点id（endpoints.beans.id=mybeans）
开启远程应用关闭功能（endpoints.shutdown.enabled=true）
关闭端点（endpoints.beans.enabled=false）
开启所需端点
endpoints.enabled=false
endpoints.beans.enabled=true
定制端点访问根路径
management.context-path=/manage
关闭http端点
management.port=-1  也就是说这些功能都访问不到了
```

### 健康监控

在SpringBoot中健康监控组件会在org.springframework.boot.actuate包下，我们也可以在IDEA上看到SpringBoot的健康状态，以Redis为例，当SpringBootMaven加入了Redis的以来之后SpringBoot会对Redis的健康状态进行监控。

![Av9MJU.png](https://s2.ax1x.com/2019/04/15/Av9MJU.png)

在SpringBoot中添加

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

![Av9ryd.png](https://s2.ax1x.com/2019/04/15/Av9ryd.png)

这是因为我们没有正确配置Redis的原因。在Docker中启动Redis，在application.properties中

```properties
spring.redis.host=192.168.1.110
```

![AvCVpD.png](https://s2.ax1x.com/2019/04/15/AvCVpD.png)

### 自定义

我们可以自定义健康状态的指示器，

首先我们要编写一个健康指示器实现HealthIndicator接口

```java
@Component
public class MyAppHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        //自定义的查询方法
        // return Health.up().build();代表健康
        return Health.down().withDetail("msg","服务器异常请排除错误").build();
    }
}
```

![AvCvUP.png](https://s2.ax1x.com/2019/04/15/AvCvUP.png)

这样没有图形界面的监控信息不是很友好我们可以在,SpringBoot的使用SpringBoot Admin

## SpringBoot Admin

### 简介

Spring Boot Admin 是一个管理和监控Spring Boot 应用程序的开源软件。每个应用都认为是一个客户端，通过HTTP或者使用 Eureka注册到admin server中进行展示，Spring Boot Admin UI部分使用AngularJs将数据展示在前端。

Spring Boot Admin 是一个针对spring-boot的actuator接口进行UI美化封装的监控工具。他可以：在列表中浏览所有被监控spring-boot项目的基本信息，详细的Health信息、内存信息、JVM信息、垃圾回收信息、各种配置信息（比如数据源、缓存列表和命中率）等，还可以直接修改logger的level。

## 准备

### server

创建SpringBoot项目初始化如下这是服务端的安装

![Av0enU.png](https://s2.ax1x.com/2019/04/16/Av0enU.png)

我们首先为了不和其他的端口发生冲突,可以配置server端口为自己的自定义的端口号这里我们配置成8000。

在SpringbootAdminServerApplication中我们添加一些注解

```java
package com.hph.server;

import de.codecentric.boot.admin.config.EnableAdminServer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Configuration;

@Configuration  //配置类
@EnableAdminServer  //开启admin服务
@SpringBootApplicatio
public class SpringbootAdminServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootAdminServerApplication.class, args);
    }

}
```

```java
package com.hph.server;

import de.codecentric.boot.admin.config.EnableAdminServer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Configuration;

@Configuration 
@EnableAdminServer  //开启admin服务
@SpringBootApplication
public class SpringbootAdminServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootAdminServerApplication.class, args);
    }
}
```

### client

我们继续创建新的Module，选择Admin的Client端

![AvBMqS.png](https://s2.ax1x.com/2019/04/16/AvBMqS.png)

在client端中我们只需要在application.properties配置上其他的不需要再进行配置。

```properties
#client占用端口
server.port=8001    

#server端的url地址
spring.boot.admin.url=http://localhost:8000\
#修改权限  
management.security.enabled=false 
```

启动client服务之后再去查看一下我们的信息，发现多了一条纪录。

![AvBoid.png](https://s2.ax1x.com/2019/04/16/AvBoid.png)

### 监控

点击Details我们进入到

#### Details

![AvD4XV.png](https://s2.ax1x.com/2019/04/16/AvD4XV.png)

#### Metrics

![AvrmB8.png](https://s2.ax1x.com/2019/04/16/AvrmB8.png)

#### Environment

![AvrmB8.png](https://s2.ax1x.com/2019/04/16/AvrmB8.png)

![Avr8cq.png](https://s2.ax1x.com/2019/04/16/Avr8cq.png)

#### JXM

![Avrsjx.png](https://s2.ax1x.com/2019/04/16/Avrsjx.png)

#### Threads

这里有一些线程信息

![Avc3IU.png](https://s2.ax1x.com/2019/04/16/Avc3IU.png)

#### Audit

审计事件

![Avg1OI.png](https://s2.ax1x.com/2019/04/16/Avg1OI.png)

#### Trace

![AvgT76.png](https://s2.ax1x.com/2019/04/16/AvgT76.png)

#### Heapdump

![AvgbtO.png](https://s2.ax1x.com/2019/04/16/AvgbtO.png)