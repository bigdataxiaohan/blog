---
title: SpringMVC概述
date: 2019-03-06 18:51:21
tags: SpringMVC
categories: JavaWeb
---

## Spring MVC简介

Spring MVC属于SpringFrameWork的后续产品，已经融合在Spring Web Flow里面。轻量级的、基于MVC的Web层应用框架。偏前端而不是基于业务逻辑层。Spring框架的一个后续产品 Spring 框架提供了构建 Web 应用程序的全功能 MVC 模块。使用Spring可插入的 MVC 架构，从而在使用Spring进行WEB开发时，可以选择使用Spring的Spring MVC框架或集成其他MVC开发框架，如[Struts](https://baike.baidu.com/item/Struts/485073)1(现在一般不用)，[Struts 2](https://baike.baidu.com/item/Struts%202/2187934)(一般老项目使用)等。

Spring MVC 通过一套 MVC 注解，让 POJO 成为处理请求的控制器，而无须实现任何接口。支持 REST 风格的 URL 请求。采用了松散耦合可插拔组件结构，比其他 MVC 框架更具扩展性和灵活性。

![kqlqrq.png](https://s2.ax1x.com/2019/03/02/kqlqrq.png)

## 功能

- 天生与Spring框架集成，如：(IOC,AOP)
- 支持Restful风格
-  进行更简洁的Web层开发
- 支持灵活的URL到页面控制器的映射
-  非常容易与其他视图技术集成(JSP HTML)，如:Velocity、FreeMarke(模板技术:商品页面页面相同)等等
- 因为模型数据不存放在特定的API里，而是放在一个Model里(Map数据结构实现，因此很容易被其他框架使用)
- 非常灵活的数据验证、格式化和数据绑定机制、能使用任何对象进行数据绑定，不必实现特定框架的API.
- 更加简单、强大的异常处理
- 对静态资源的支持
-  支持灵活的本地化、主题等解析

## 使用  

 SpringMVC将Web层进行了职责解耦，基于请求-响应模型

常用的组件

| 组件                     | 功能                                                         |
| ------------------------ | ------------------------------------------------------------ |
| DispatcherServlet        | 前端控制器                                                   |
| Controller               | 处理器/页面控制器，做的是MVC中的C的事情，但控制逻辑转移到前端控制器了，用于对请求进行处理 |
| HandlerMapping           | 请求映射到处理器，找谁来处理，如果映射成功返回一个HandlerExecutionChain对象（包含一个Handler处理器(页面控制器)对象、多个HandlerInterceptor拦截器对象） |
| View<br/>Resolver        | 视图解析器，找谁来处理返回的页面。把逻辑视图解析为具体的View,进行这种策略模式，很容易更换其他视图技术    如InternalResourceViewResolver将逻辑视图名映射为JSP视图 |
| LocalResolver            | 本地化、国际化                                               |
| MultipartResolver        | 文件上传解析器                                               |
| HandlerExceptionResolver | 异常处理器                                                   |

## 环境配置

![kvdX6I.png](https://s2.ax1x.com/2019/03/06/kvdX6I.png)

目录结构如下

![kvwd4e.png](https://s2.ax1x.com/2019/03/06/kvwd4e.png)

配置Tomcat

![kvwB3d.png](https://s2.ax1x.com/2019/03/06/kvwB3d.png)

这里出现了问题我们进行修复以下

![kvwRUS.png](https://s2.ax1x.com/2019/03/06/kvwRUS.png)

点击第二个选项

![kvwv8J.png](https://s2.ax1x.com/2019/03/06/kvwv8J.png)

基本完成修复

运行Tomcat

![kv0Cb6.png](https://s2.ax1x.com/2019/03/06/kv0Cb6.png)

环境配置完成

## HelloWorld

![kvsapF.png](https://s2.ax1x.com/2019/03/06/kvsapF.png)

### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>springDispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!--
        load-on-startup:设置DispathcerServlet
            Servlet的创建时机:
                   请求到达以后创建:
                   服务器启动即创建:
        -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springDispatcherServlet</servlet-name>
        <!--任何请求都会进去 对于JSP请求 不会处理-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

### springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--组件扫描-->
    <context:component-scan base-package="com.hph.springmvc"></context:component-scan>

    <!--视图解析器：
           工作机制：prefix +请求方式的返回值 +suffix = 物理视图路径
                   WEB-INF/views/success.jsp
             WEB-INF:是服务器内部路径，不能从浏览器访问该l路径下的资源，但是可以内部转发进行访问

    -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```

### index.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: Schindler
  Date: 2019/3/6
  Time: 20:28
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<a href="springmvc/testPathVariable/admin/1001">Test PathVariable</a>
<br>
<a href="springmvc/testRequestMappingParamsAndHeaders?username=Tom&age=22">Test RequestMapping Parms Headers</a>
<br/>
<a href="springmvc/testRequestMapping">Hello SpringMVC</a>
<br/>
<a href="springmvc/testMethord">TestMethord</a>

</body>
</html>

```

### SpringMVCHandler

```java
package com.hph.springmvc.helloWorld;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
@RequestMapping("/springmvc")
public class SpringMVCHandler {
    /**
     * 带占位符的URL
     * 浏览器： http://localhost/SpringMVC//testPathVariable/admin/1001
     */
    @RequestMapping(value = "/testPathVariable/{name}/{id}")
    public String testPathVariable(@PathVariable("name") String name, @PathVariable("id") Integer id) {
        System.out.println(name + ":" + id);
        return "success";

    }

    /**
     * @RequestMapping 映射请求参数 以及 请求头信息
     * params : name=Tom&age=22
     * headers
     */
    @RequestMapping(value = "/testRequestMappingParamsAndHeaders", params = {"username", "age=22"}, headers = {"Accept-Language"})
    public String testRequestMappingParamsAndHeaders() {
        return "success";
    }

    @RequestMapping(value = "/testMethord", method = RequestMethod.POST)
    public String testMethord() {
        System.out.println("testMethord...");
        return "success";
    }

    @RequestMapping("/testRequestMapping")
    public String testRequestMapping() {
        return "success";
    }
}


```

### 运行结果

![kvyv8O.png](https://s2.ax1x.com/2019/03/06/kvyv8O.png)

![kvLmbq.png](https://s2.ax1x.com/2019/03/07/kvLmbq.png)

![kvLvJU.png](https://s2.ax1x.com/2019/03/07/kvLvJU.png)

### 运行分析

![kvyuAx.png](https://s2.ax1x.com/2019/03/06/kvyuAx.png)

![kvyrvQ.png](https://s2.ax1x.com/2019/03/06/kvyrvQ.png)

### 基本步骤

①    客户端请求提交到**DispatcherServlet**

②    由DispatcherServlet控制器查询一个或多个**HandlerMapping**，找到处理请求的**Controller**

③    DispatcherServlet将请求提交到Controller（也称为Handler）

④    Controller调用业务逻辑处理后，返回**ModelAndView**

⑤    DispatcherServlet查询一个或多个**ViewResoler**视图解析器，找到ModelAndView指定的视图

⑥    视图负责将结果显示到客户端

## @RequestMapping

- SpringMVC使用@RequestMapping注解为控制器指定可以处理哪些 URL 请求
- 在控制器的**类定义及方法定义处**都可标注 @RequestMapping
    -  **标记在类上**：提供初步的请求映射信息。相对于  WEB 应用的根目录
    - **标记在方法上**：提供进一步的细分映射信息。相对于标记在类上的 URL。
- 若类上未标注 @RequestMapping，则方法处标记的 URL 相对于 WEB 应用的根目录 
-  作用：DispatcherServlet 截获请求后，就通过控制器上 @RequestMapping 提供的映射信息确定请求所对应的处理方法。 

```java
Target({ElementType.METHOD, ElementType.TYPE})	//Target来标注当前注解标注的位置   方法 类
@Retention(RetentionPolicy.RUNTIME)			
@Documented
@Mapping
public @interface RequestMapping {
    String name() default "";				  //默认可以省略value 

    @AliasFor("path")
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};	//映射请求方式  GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE

    String[] params() default {};

    String[] headers() default {};

    String[] consumes() default {};

    String[] produces() default {};
}

```

## REST

### 资料链接

理解本真的REST架构风格: <http://kb.cnblogs.com/page/186516/> 

REST: <http://www.infoq.com/cn/articles/rest-introduction>

### web.xml

```xml
    <!--配置REST 过滤器 HiddenHttpMethod-->
    <filter>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

### SpringMVCHandler

```java
 @RequestMapping(value="/testRESTGet/{id}",method=RequestMethod.GET)
    public String testRESTGet(@PathVariable(value="id") Integer id){
        System.out.println("testRESTGet id="+id);
        return "success";
    }

    @RequestMapping(value="/testRESTPost",method=RequestMethod.POST)
    public String testRESTPost(){
        System.out.println("testRESTPost");
        return "success";
    }

    @RequestMapping(value="/testRESTPut/{id}",method=RequestMethod.PUT)
    public String testRESTPut(@PathVariable("id") Integer id){
        System.out.println("testRESTPut id="+id);
        return "success";
    }

    @RequestMapping(value="/testRESTDelete/{id}",method=RequestMethod.DELETE)
    public String testRESTDelete(@PathVariable("id") Integer id){
        System.out.println("testRESTDelete id="+id);
        return "success";
    }
```



![kx9Kaj.png](https://s2.ax1x.com/2019/03/07/kx9Kaj.png)







