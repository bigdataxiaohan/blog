---
title: SpringCloud与zuul
date: 2019-04-27 17:24:52
categories: 微服务
tags:
- SpringCloud
- zuul
---

## 简介

Zuul包含了对请求的路由和过滤两个最主要的功能：
其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

注意：Zuul服务最终还是会注册进Eureka

提供=代理+路由+过滤三大功能

## 步骤

### 创建microservicecloud-zuul-gateway-9527

#### pom.xml

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

    <artifactId>microservicecloud-zuul-gateway-9527</artifactId>
    <dependencies>
        <!-- zuul路由网关 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <!-- actuator监控 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--  hystrix容错-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <!-- 日常标配 -->
        <dependency>
            <groupId>com.hph.springcloud</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!-- 热部署插件 -->
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

#### application.yml

```yaml
server:
  port: 9527

spring:
  application:
    name: microservicecloud-zuul-gateway

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true


info:
  app.name: hph-microservicecloud
  company.name: www.hphblog.cn
  build.artifactId: ${project.artifactId}
  build.version: ${project.version}
```

#### hosts修改

```java
127.0.0.1  myzuul.com
```

#### Zuul_9527_StartSpringCloudApp

```java
package com.hph.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableZuulProxy
public class Zuul_9527_StartSpringCloudApp
{
  public static void main(String[] args)
  {
   SpringApplication.run(Zuul_9527_StartSpringCloudApp.class, args);
  }
}
```

## 测试

1. 启动三个eurekaserver集群。
2. 启动microservicecloud-provider-dept-8001类。
3. 启动Zuul_9527_StartSpringCloudApp。
4. 不使用路由访问&nbsp;<http://localhost:8001/dept/get/2>

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Hystrix/20190427173636.png)

5. 使用路由访问&nbsp;http://myzuul.com:9527/microservicecloud-dept/dept/get/2

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Hystrix/20190427173803.png)

## 添加映射

### application.yml

添加

```yaml
zuul: 
  routes: 
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
```

访问：<http://myzuul.com:9527/mydept/dept/get/1>

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Hystrix/20190427174336.png)

但是这个路径还存在我们可以屏蔽一下。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Hystrix/20190427174614.png)

```yaml
zuul:
  ignored-services: microservicecloud-dept
  routes:
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Hystrix/20190427175033.png)

如果我们想批量的屏蔽真实的地址我们可以使用`“*”`

```yaml
zuul:
  ignored-services: "*"
  routes:
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
```

设置同一的前缀安全加固

```yaml
zuul: 
  prefix: /hphblog
  ignored-services: "*"
  routes: 
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
 
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/Hystrix/20190427175609.png)

## 完整代码

Github: <https://github.com/bigdataxiaohan/microservicecloud/tree/master/zuul>



