---
title: Spring和SpringMVC整合
date: 2019-03-13 19:47:56
tags: SpringMVC
categories: JavaWeb
---

## 原因

SpringMVC就运行在Spring环境之下，为什么还要整合呢？SpringMVC和Spring都有IOC容器，是不是都需要保留呢？

通常情况下，类似于数据源，事务，整合其他框架都是放在spring的配置文件中（而不是放在SpringMVC的配置文件中）,实际上放入Spring配置文件对应的IOC容器中的还有Service和Dao.而SpringMVC也搞自己的一个IOC容器，在SpringMVC的容器中只配置自己的Handler(Controller)信息。所以，两者的整合是十分有必要的，SpringMVC负责接受页面发送来的请求，Spring框架则负责整理中间需求逻辑，对数据库发送操作请求，对数据库的操作目前则先使用Spring框架中的JdbcTemplate进行处理。

## 目录结构

![AkUV8P.png](https://s2.ax1x.com/2019/03/13/AkUV8P.png)

### 详细结构

#### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--spring的配置文件-->
    <bean id="person" class="com.hph.ss.beans.Person">
        <property name="name" value="Spring+SpringMVC"></property>
    </bean>
    <!-- 组件扫描 -->
 <context:component-scan base-package="com.hph.ss"></context:component-scan>
</beans>
```

#### Person

```java
package com.hph.ss.beans;

public class Person {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void sayHello() {
        System.out.println("My name is " + name);
    }

}
```

#### UserDao

```java
package com.hph.ss.dao;

import org.springframework.stereotype.Repository;

@Repository
public class UserDao {
    public UserDao(){
        System.out.println("UserDao....");
    }
    public void hello() {
        System.out.println("UserDao  hello.....");
    }
}

```

#### UserHandler

```java
package com.hph.ss.handler;

import org.springframework.stereotype.Controller;

@Controller
public class UserHandler {

    public UserHandler() {
        System.out.println("UserHandler.......");
    }
}
```

#### MyServletContextlistener

```java
package com.hph.ss.listerner;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import javax.servlet.ServletContext;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;

public class MyServletContextlistener implements ServletContextListener {
    /**
     * 当监听到ServletContext被创建,则执行该方法
     */
    public void contextInitialized(ServletContextEvent sce) {
        //1.创建SpringIOC容器对象
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

        //2.将SpringIOC容器对象绑定到ServletContext中
        ServletContext sc = sce.getServletContext();

        sc.setAttribute("applicationContext", ctx);
    }

    public void contextDestroyed(ServletContextEvent sce) {

    }
}

```

#### UserService

```java
package com.hph.ss.service;

import com.hph.ss.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {
	
	@Autowired
	private UserDao userDao  ;
	
	public UserService() {
		System.out.println("UserService ......");
	}
	
	public void hello() {
		userDao.hello();
	}
}
```

#### HelloServlet

```java
package com.hph.ss.servlet;

import com.hph.ss.beans.Person;
import org.springframework.context.ApplicationContext;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "HelloServlet")
public class HelloServlet extends HttpServlet {

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //访问到SpringIOC容器中的person对象.
        //从ServletContext对象中获取SpringIOC容器对象
        ServletContext sc = getServletContext();

        ApplicationContext ctx = (ApplicationContext) sc.getAttribute("applicationContext");

        Person person = ctx.getBean("person", Person.class);
        person.sayHello();

    }
}
```

#### springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!--组件扫描-->
    <context:component-scan base-package="com.hph.ss
"></context:component-scan>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="WEB-INF/views/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
    <!--处理静态资源-->
    <mvc:default-servlet-handler/>
    <mvc:annotation-driven/>
</beans>
```

### web

#### index.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
  <a href="hello">Hello Springmvc</a>
  </body>
</html>
```

#### web. xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 初始化SpringIOC容器的监听器 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Springmvc的前端控制器 -->
    <!-- The front controller of this Spring Web application, responsible for handling all application requests -->
    <servlet>
        <servlet-name>springDispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- Map all requests to the DispatcherServlet for handling -->
    <servlet-mapping>
        <servlet-name>springDispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```



### 问题

![AkwIi9.png](https://s2.ax1x.com/2019/03/13/AkwIi9.png)

原因:

#### 问题描述

Spring集成SpringMVC启动后同一个bean注入了两次

#### 原因分析

Sping+SpringMVC的框架中，IoC容器的加载过程：

1. 基本上Web容器(如Tomcat)先加载ContextLoaderListener，然后生成一个IoC容器。
2. 然后再实例化DispatchServlet时候会加载对应的配置文件，再次生成Controller相关的IoC容器。

关于上面两个容器关系：

ContextLoaderListener中创建ApplicationContext主要用于整个Web应用程序需要共享的一些组件，比如DAO，数据库的ConnectionFactory等。而由**DispatcherServlet**创建的ApplicationContext主要用于和该Servlet相关的一些组件，比如Controller、ViewResovler等。

对于作用范围而言，在DispatcherServlet中可以引用由ContextLoaderListener所创建的ApplicationContext，而反过来不行。

### 解决方法

#### 方法1

springmvc.xml

```xml
<!--组件扫描-->
<context:component-scan base-package="com.hph.ss.handler"></context:component-scan>
```

applicationContext.xml

```xml
<context:component-scan base-package="com.hph.ss.dao,com.hph.ss.service"></context:component-scan>
```

![Ak0MQ0.png](https://s2.ax1x.com/2019/03/13/Ak0MQ0.png)

#### 方法2

springmvc.xml

```
   <!--组件扫描-->
    <context:component-scan base-package="com.hph.ss" use-default-filters="false">
        <context:include-filter type="annotation"
                                expression="org.springframework.stereotype.Controller"></context:include-filter>
    </context:component-scan>
```

applicationContext.xml

```xml
    <!-- 组件扫描 -->
 <context:component-scan base-package="com.hph.ss">
     <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"></context:exclude-filter>
 </context:component-scan>
```

![Akyk5D.png](https://s2.ax1x.com/2019/03/13/Akyk5D.png)

##  关系

servlet:代表的的容器为spring-mvc的子容器，而DispatcherServlet 是前端控制器，该容器专门为前端监听请求的时候所用，就是说当接收到url请求的时候会引用springmvc容器内的对象来处理。

context-param:代表的容器是spring本身的容器，spring-mvc可以理解为一个继承自该容器的子容器，spring容器是最顶层的父类容器，跟java的继承原理一样，子容器能使用父类的对象，但是父容器不能使用子类的对象

初始化的顺序也是父类容器优先级高，当服务器解析web.xml的时候由于listener监听的原因，会优先初始化spring容器，之后才初始化spring-mvc容器。

 在 Spring MVC 配置文件中引用业务层的 Bean

多个 Spring IOC 容器之间可以设置为父子关系，以实现良好的解耦。

Spring MVC WEB 层容器可作为 “业务层” Spring 容器的子容器：即 WEB 层容器可以引用业务层容器的 Bean，而业务层容器却访问不到 WEB 层容器的 Bean

![Ak2eFf.png](https://s2.ax1x.com/2019/03/13/Ak2eFf.png)





