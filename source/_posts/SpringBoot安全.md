---
title: SpringBoot与安全
date: 2019-04-13 15:43:34
tags: Spring Security
categories: SpringBoot 
---

## 简介

### 安全框架

Spring Security是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型。他可以实现强大的web安全控制。对于安全控制，我们仅需引入spring-boot-starter-security模块，进行少量的配置，即可实现强大的安全管理。

Apache Shiro是一个强大且易用的Java安全框架,执行身份验证、授权、密码和会话管理。使用Shiro的易于理解的API,您可以快速、轻松地获得任何应用程序,从最小的移动应用程序到最大的网络和企业应用程序。

### Spring Security

```java
WebSecurityConfigurerAdapter：自定义Security策略
AuthenticationManagerBuilder：自定义认证策略
@EnableWebSecurity：开启WebSecurity模式
```

应用程序的两个主要区域是“认证”和“授权”（或者访问控制）。这两个主要区域是Spring Security 的两个目标。

“认证”（Authentication），是建立一个他声明的主体的过程（一个“主体”一般是指用户，设备或一些可以在你的应用程序中执行动作的其他系统）。

“授权”（Authorization）指确定一个主体是否允许在你的应用程序执行一个动作的过程。为了抵达需要授权的店，主体的身份已经有认证过程建立。

这个概念是通用的而不只在Spring Security中。

## 准备

创建一个项目Springboot为1.5.20,并且导入thymeleaf支持。

### 页面准备

链接：https://pan.baidu.com/s/11UWsVj5rohGms24Xl8rFYQ  提取码：3pwj 

### Controller

```java
package com.hph.springbootsecurity.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Controller
public class KungfuController {
	private final String PREFIX = "pages/";
	/**
	 * 欢迎页
	 * @return
	 */
	@GetMapping("/")
	public String index() {
		return "welcome";
	}
	
	/**
	 * 登陆页
	 * @return
	 */
	@GetMapping("/userlogin")
	public String loginPage() {
		return PREFIX+"login";
	}
	
	
	/**
	 * level1页面映射
	 * @param path
	 * @return
	 */
	@GetMapping("/level1/{path}")
	public String level1(@PathVariable("path")String path) {
		return PREFIX+"level1/"+path;
	}
	
	/**
	 * level2页面映射
	 * @param path
	 * @return
	 */
	@GetMapping("/level2/{path}")
	public String level2(@PathVariable("path")String path) {
		return PREFIX+"level2/"+path;
	}
	
	/**
	 * level3页面映射
	 * @param path
	 * @return
	 */
	@GetMapping("/level3/{path}")
	public String level3(@PathVariable("path")String path) {
		return PREFIX+"level3/"+path;
	}

}
```

![ALGYlR.png](https://s2.ax1x.com/2019/04/13/ALGYlR.png)

启动之后报错这是因为我们的thymeleaf版本不支持，因此我们需要更改一下pom文件中的配置信息。

```xml
    <properties>
        <java.version>1.8</java.version>
        <thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
        <thymeleaf-layout-dialect.version>2.3.0</thymeleaf-layout-dialect.version>
    </properties>
```

![ALJGE8.png](https://s2.ax1x.com/2019/04/13/ALJGE8.png)

现在这套武当派的秘籍管理系统不是很好我们需要完善一下。

## 步骤

### 引入Spring Security

在pom文件中添加

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### Spring Security的配置

SpringBoot帮助我们配置了大多数的Spring Security，因此我们只需要编写一个配置类即可。参考[官网](<https://docs.spring.io/spring-security/site/docs/current/guides/html5/helloworld-boot.html>)

```java
package com.hph.springbootsecurity.config;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@EnableWebSecurity
//编写SpringSecurity的配置类
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //注销掉父类的默认规则
        //super.configure(http);

        //定制请求的授权规则
        http.authorizeRequests().antMatchers("/").
                permitAll()
                .antMatchers("/level1/**").hasRole("newbee")
                .antMatchers("/level2/**").hasRole("senior")
                .antMatchers("/level3/**").hasRole("master");
    }
}
```

在配置之后我们先看一下效果。

![ALtAYD.png](https://s2.ax1x.com/2019/04/13/ALtAYD.png)

主页可以访问不过其他的组件时候。不可以访问。提示403，必须角色相互匹配。

![ALteld.png](https://s2.ax1x.com/2019/04/13/ALteld.png)

当我们开启自动配置登录的时候

```java
//开启自动配置的登录功能
 http.formLogin();
//1.login请求来到登录页
//2.如果登录错误,重定向到/login?error
```



![ALtvAf.png](https://s2.ax1x.com/2019/04/13/ALtvAf.png)

### 自定义规则 

开发过程中尽量不要使用中文

```java
  //定义认证规则
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
     //   super.configure(auth);
        auth.inMemoryAuthentication()
                .withUser("武当侠士").password("cainiao").roles("newbee")
                .and()
                .withUser("无为真人").password("zhongji").roles("newbee","senior")
                .and()
                .withUser("武当天尊").password("dalao").roles("newbee","senior","master");
    }
```

经过测试相应角色都可以访问。

### 添加注销

首先我们在welcome.html中添加

```html
<h1 align="center">欢迎光临武当秘籍管理系统</h1>
<h2 align="center">游客您好，如果想查看武当秘籍 <a th:href="@{/login}">请成为武当弟子</a></h2>
<!--添加-->
<form th:action="@{/logout}" method="post">
	<input type="submit" value="注销"/>
</form>
```
配置类中设置logout;
```java
http.logout();
```

注销成功之后会返回login?logout的登录页，当然我们也可以定制url

![ALwSZn.png](https://s2.ax1x.com/2019/04/13/ALwSZn.png)

定制注销后返回的url

```java
http.logout().logoutSuccessUrl("/");
```

如何让游客显示为特定角色呢？我们需要引入

```xml
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
        <thymeleaf-layout-dialect.version>2.3.0</thymeleaf-layout-dialect.version>
        <thymeleaf-extras-springsecurity4.version>3.0.2.RELEASE</thymeleaf-extras-springsecurity4.version>
    </properties>

    <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity4</artifactId>
        </dependency>
```

### 显示角色权限

在welcome.html中添加

```html
<!--没有认证的情况下-->
<div sec:authorize="!isAuthenticated()">
<h2 align="center">游客您好，如果想查看武当秘籍 <a th:href="@{/login}">请成为武当弟子</a></h2>
</div>
<!--认证了-->
<div sec:authorize="isAuthenticated()">
	<h2><span sec:authentication="name"></span>，您好,您的角色有：
		<span sec:authentication="principal.authorities"></span></h2>
	<form th:action="@{/logout}" method="post">
		<input type="submit" value="注销"/>
	</form>
</div>
```



![AL0LuQ.png](https://s2.ax1x.com/2019/04/13/AL0LuQ.png)

![AL6I4s.png](https://s2.ax1x.com/2019/04/13/AL6I4s.png)

### 对应权限显示

我们需要在welcome.html中修改

```html
<div sec:authorize="hasRole('newbee')">
    <h3>普通武功秘籍</h3>
    <ul>
        <li><a th:href="@{/level1/1}">八卦掌</a></li>
        <li><a th:href="@{/level1/2}">犀牛望月</a></li>
        <li><a th:href="@{/level1/3}">太渊十三剑</a></li>
    </ul>
</div>

<div sec:authorize="hasRole('senior')">
    <h3>高级武功秘籍</h3>
    <ul>
        <li><a th:href="@{/level2/1}">梯云纵</a></li>
        <li><a th:href="@{/level2/2}">七星聚首</a></li>
        <li><a th:href="@{/level2/3}">天外飞仙</a></li>
    </ul>
</div>

<div sec:authorize="hasRole('master')">
    <h3>绝世武功秘籍</h3>
    <ul>
        <li><a th:href="@{/level3/1}">神照经</a></li>
        <li><a th:href="@{/level3/2}">九阴真经</a></li>
        <li><a th:href="@{/level3/3}">独孤九剑</a></li>
    </ul>
</div>
```

### 记住我功能

在MySecurityConfig中添加

```java
//开启记住我功能
http.rememberMe();
```

开启之后再次登录就有一个记住我的按钮了

![ALgiJs.png](https://s2.ax1x.com/2019/04/13/ALgiJs.png)

![ALgTXV.png](https://s2.ax1x.com/2019/04/13/ALgTXV.png)

登录成功之后cookie发送给浏览器保存，以后登录带上这个cookie，只要通过检查就可以免登录，如果点击注销会删除这个cookie

### 定制登录页

在MySecurityConfig中使用

```java
http.formLogin().usernameParameter("user").passwordParameter("passwd").loginPage("/userlogin");
```

在修改welcome.html 发送请求为userlogin

```html
<h2 align="center">游客您好，如果想查看武当秘籍请<a th:href="@{/userlogin}">成为武当弟子</a></h2>
```

默认post形式的/login代表处理登录，但是如果一旦定制了loginPage的post请求就是登录

login.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h1 align="center">欢迎登录武当秘籍管理系统</h1>
	<hr>
	<div align="center">
		<form th:action="@{/userlogin}" method="post">
			用户名:<input name="user"/><br>
			密&nbsp;&nbsp;&nbsp;码:<input name="passwd"><br/>
			<input type="submit" value="登陆">
		</form>
	</div>
</body>
</html>
```

![ALheIS.png](https://s2.ax1x.com/2019/04/13/ALheIS.png)

实现了登录界面的跳转。

#### 添加记住我功能

在MySecurityConfig中添加

```java
//开启记住我功能
 http.rememberMe().rememberMeParameter("remeber");
```

在userlogin.html中添加

```html
<div align="center">
    <br th:action="@{/userlogin}" method="post">
        用户名:<input name="user"/><br>
        密&nbsp;&nbsp;&nbsp;码:<input name="passwd"><br/>
        <!--添加记住我按钮-->
        <input type="checkbox" name="remeber">记住我</br>
        <input type="submit" value="登陆">
    </form>
</div>
```

![ALh5Lt.png](https://s2.ax1x.com/2019/04/13/ALh5Lt.png)

没有提交表单之前式没有rember这个cookie的

![AL4FW4.png](https://s2.ax1x.com/2019/04/13/AL4FW4.png)

这样下次就不用手动输入密码了。







