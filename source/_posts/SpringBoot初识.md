---
title: SpringBoot初识
date: 2019-03-25 12:06:11
tags: SpringBoot
categories: SpringBoot
---

## 简介

Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。通过这种方式，Spring Boot致力于在蓬勃发展的快速应用开发领域(rapid application development)成为领导者。

## 原因

随着动态语言的流行（Ruby、Groovy、Scala、Node.js)，Java的幵发显得格外的笨重：繁多的配置、低下的开发效率、复杂的部署流程以及第三方技术集成难度大。在上述环境下，Spring Boot应运而生。它使用“习惯优于配置”（项目中存在大量的配置，此外还内置一个习惯性的配置，让你无须手动进行配置）的理念让你的项目快速运行起来。使用Spring Boot很容易创建一个独立运行(运行jar,内嵌Servlet容器）、准生产级别的基于Spring框架的项目，使用Spring Boot你可以不用或者只需要很少的Spring配置。

## 特点

创建独立的Spring应用程序:Spring Boot可以以jar包的形式独立运行，运行一个Spring Boot项目只需通过java -jar xx.jar来运行。

嵌入的Tomcat，无需部署WAR文件:Spring Boot可选择内嵌Tomcat、Jetty或者Undertow ,这样我们无须以war包形式部署项目。

简化Maven配置:Spring提供了一系列的starter pom来简化Maven的依赖加载，spring-boot-starter-web

自动配置Spring:Spring Boot会根据在类路径中的jar包、类，为jar包里的类自动配置Bean，.这样会极大地减少我们要使用的配置。当然,Spring Boot只是考虑了大多数的开发场景，并不是所有的场景，若在实际开发中我们需要自动配置Bean，而Spring Boot没有提供支持，则可以自定义自动配置

提供生产就绪型功能，如指标，健康检查和外部配置:Spring Boot提供基于http、ssh、telnet对运行时的项目进行监控

绝对没有代码生成并且对XML也没有配置要求:Spring Boot的神奇的不是借助于代码生成来实现的，而是通过条件注解来实现的，这是 Spring 4.x提供的新特性，Spring 4.x提倡使用Java配置和注解配置组合，而Spring Boot不需要任何xml配置即可实现Spring的所有配置.

## 单体应用

![AtK9tx.png](https://s2.ax1x.com/2019/03/25/AtK9tx.png)

## 微服务

![AtKZBd.png](https://s2.ax1x.com/2019/03/25/AtKZBd.png)

[微服务文档](https://martinfowler.com/articles/microservices.html#MicroservicesAndSoa)

通信(Http)

![AtK1gS.png](https://s2.ax1x.com/2019/03/25/AtK1gS.png)

![AtKNEn.png](https://s2.ax1x.com/2019/03/25/AtKNEn.png)

## 优点

- 快速创建独立运行的Spring项目以及与主流框架集成 
- 使用嵌入式的Servlet容器，应用无需打成WAR包 
- starters自动依赖与版本控制 
- 大量的自动配置，简化开发，也可修改默认值 
- 无需配置XML，无代码生成，开箱即用 
- 准生产环境的运行时应用监控 – 与云计算的天然集成

## 缺点

- 将现有或传统的Spring Framework项目转换为Spring Boot应用程序是一个非常困难和耗时的过程。它仅适用于全新Spring项目；
- 集成度较高，使用过程中不太容易了解底层；

## HelloWorld

初始化项目

![AtuYY6.png](https://s2.ax1x.com/2019/03/25/AtuYY6.png)

2.设置

![AtualD.png](https://s2.ax1x.com/2019/03/25/AtualD.png)

3.选中web组件

![Atud6e.png](https://s2.ax1x.com/2019/03/25/Atud6e.png)

3.下一步

![AtuwOH.png](https://s2.ax1x.com/2019/03/25/AtuwOH.png)

4.项目结构

![AtufXQ.png](https://s2.ax1x.com/2019/03/25/AtufXQ.png)

添加ava类

```java
package com.hph.springboot.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;


@Controller
public class HelloController {
    @RequestMapping("/helloworld")
    public  @ResponseBody String hello(){
        return "Hello Spring Boot ";

    }
}
```

![AtuOcF.png](https://s2.ax1x.com/2019/03/25/AtuOcF.png)

启动服务

![AtuxB9.png](https://s2.ax1x.com/2019/03/25/AtuxB9.png)

## 部署项目

如果没有SpringBoot的Maven插件 则需要在pom.xml加入

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```



### 打包

![AtKR4x.png](https://s2.ax1x.com/2019/03/25/AtKR4x.png)

### 测试

![AtKTDH.png](https://s2.ax1x.com/2019/03/25/AtKTDH.png)

## 分析

为什么我们什么也没有配置 就起来了一个JavaEE项目呢.

### 父项目

父项目在传统开发中做依赖管理,在SpringBoot中

```xml
  <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/>

   
<!--Spring Boot Dependencies管理应用里的所有依赖版本-->
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.1.3.RELEASE</version>
    <relativePath>../spring-boot-dependencies</relativePath>
  </parent>
```

SpringBoot的版本的仲裁中心：

所有的版本依赖都在 dependencies 里面管理（如果没有在dependenciesl里面的依赖要声明版本号）

### 导入依赖

```xml
    <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

spring-boot-starter-web:

spring-boot-starter-web:spring-boot场景启动器，点进去发现springboot场景启帮我们导入了webmo模块正常运行的所依赖的组件。

Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器

```xml
<dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>2.1.3.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-json</artifactId>
      <version>2.1.3.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <version>2.1.3.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.hibernate.validator</groupId>
      <artifactId>hibernate-validator</artifactId>
      <version>6.0.14.Final</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.1.5.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.1.5.RELEASE</version>
      <scope>compile</scope>
    </dependency>
  </dependencies>
```

### 主程序

```java
package com.hph.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloworldApplication {

    public static void main(String[] args) {
        SpringApplication.run(HelloworldApplication.class, args);
    }

}

```

## 自动配置

如果我们注释掉主程序类启动则会

![AtQalT.png](https://s2.ax1x.com/2019/03/25/AtQalT.png)

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};
}

```

注解源码主要组合了©Configuration、@EnableAutoConfiguration、@ComponentScan ；

![AtQq9P.png](https://s2.ax1x.com/2019/03/25/AtQq9P.png)

@SpringBootApplication :Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法启动SpringBoot应用

@**SpringBootConfiguration**:Spring Boot的配置类:标注在某个类上，表示这是一个Spring Boot的配置类；

@**Configuration**:配置类上来标注这个注解；配置类 -----  配置文件；配置类也是容器中的一个组件；@Component

@**EnableAutoConfiguration**：开启自动配置功能；以前我们需要配置的东西，Spring Boot帮我们自动配置；@**EnableAutoConfiguration**告诉SpringBoot开启自动配置功能；这样自动配置才能生效；

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```

@AutoConfigurationPackage自动配置包

@Import({AutoConfigurationImportSelector.class})
Spring的底层注解@import，给容器导入了一个组件；组件由AutoConfigurationPackages.Registrar.class；

将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器 
@**Import**(EnableAutoConfigurationImportSelector.class)；

给容器中导入组件：

**EnableAutoConfigurationImportSelector**：导入哪些组件的选择器；

将所有需要导入的组件以全类名的方式返回；这些组件就会被添加到容器中；

会给容器中导入非常多的自动配置类（xxxAutoConfiguration）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件；

Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作；以前我们需要自己配置的东西，自动配置类都帮我们；



