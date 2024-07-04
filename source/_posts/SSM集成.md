---
title: SSM集成
date: 2019-03-24 15:09:30
tags: SSM
categories: JavaWeb
---

## 简介

SSM（Spring+SpringMVC+MyBatis）框架集由Spring、MyBatis两个开源框架整合而成（SpringMVC是Spring中的部分内容）。常作为数据源较简单的web项目的框架。

### Spring
　　Spring就像是整个项目中装配bean的大工厂，在配置文件中可以指定使用特定的参数去调用实体类的构造方法来实例化对象。也可以称之为项目中的粘合剂。
　　Spring的核心思想是IoC（控制反转），即不再需要程序员去显式地`new`一个对象，而是让Spring框架帮你来完成这一切。
### SpringMVC
　　SpringMVC在项目中拦截用户请求，它的核心Servlet即DispatcherServlet承担中介或是前台这样的职责，将用户请求通过HandlerMapping去匹配Controller，Controller就是具体对应请求所执行的操作。SpringMVC相当于SSH框架中struts。
### mybatis
　　mybatis是对jdbc的封装，它让数据库底层操作变的透明。mybatis的操作都是围绕一个sqlSessionFactory实例展开的。mybatis通过配置文件关联到各实体类的Mapper文件，Mapper文件中配置了每个类对数据库所需进行的sql语句映射。在每次与数据库交互时，通过sqlSessionFactory拿到一个sqlSession，再执行sql命令。

页面发送请求给控制器，控制器调用业务层处理逻辑，逻辑层向持久层发送请求，持久层与数据库交互，后将结果返回给业务层，业务层将处理逻辑发送给控制器，控制器再调用视图展现数据

## 创建项目

首先我们先创建Maven项目

![AY8qTH.png](https://s2.ax1x.com/2019/03/24/AY8qTH.png)

![AYGS6f.png](https://s2.ax1x.com/2019/03/24/AYGS6f.png)

![AYGP0g.png](https://s2.ax1x.com/2019/03/24/AYGP0g.png)

项目结构如下

![AYGkkj.png](https://s2.ax1x.com/2019/03/24/AYGkkj.png)

### Maven依赖

```xml
<dependencies>                                                              
    <dependency>                                                            
        <groupId>javax.servlet</groupId>                        
        <artifactId>servlet-api</artifactId>                          
        <version>2.5</version>                              
    </dependency>                                                           
    <dependency>                                                            
        <groupId>javax.servlet.jsp</groupId>                        
        <artifactId>jsp-api</artifactId>                       
        <version>2.1.3-b06</version>                              
    </dependency>                                                           
    <dependency>                                                            
        <groupId>org.springframework</groupId>                        
        <artifactId>spring-core</artifactId>                          
        <version>4.0.0.RELEASE</version>                              
    </dependency>                                                           
    <dependency>                                                            
        <groupId>org.springframework</groupId>                        
        <artifactId>spring-context</artifactId>                       
        <version>4.0.0.RELEASE</version>                              
    </dependency>                                                           
    <dependency>                                                            
        <groupId>org.springframework</groupId>                        
        <artifactId>spring-jdbc</artifactId>                          
        <version>4.0.0.RELEASE</version>                              
    </dependency>                                                           
    <dependency>                                                            
        <groupId>org.springframework</groupId>                        
        <artifactId>spring-orm</artifactId>                           
        <version>4.0.0.RELEASE</version>                              
    </dependency>                                                           
    <dependency>                                                            
        <groupId>org.springframework</groupId>                        
        <artifactId>spring-web</artifactId>                           
        <version>4.0.0.RELEASE</version>                              
    </dependency>                                                           
    <dependency>                                                            
        <groupId>org.springframework</groupId>                        
        <artifactId>spring-webmvc</artifactId>                        
        <version>4.0.0.RELEASE</version>                              
    </dependency>                                                           
    <dependency>                                                            
        <groupId>com.mchange</groupId>                                
        <artifactId>c3p0</artifactId>                                 
        <version>0.9.2</version>                                      
    </dependency>                                                           
    <dependency>                                                            
        <groupId>cglib</groupId>                                      
        <artifactId>cglib</artifactId>                                
        <version>2.2</version>                                        
    </dependency>                                                           
    <dependency>                                                            
        <groupId>org.aspectj</groupId>                                
        <artifactId>aspectjweaver</artifactId>                        
        <version>1.6.8</version>                                      
    </dependency>                                                           
                                                                                  
    <!-- Spring整合MyBatis -->                                              
    <!-- MyBatis中延迟加载需要使用Cglib -->                                 
    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->         
    <dependency>                                                            
        <groupId>org.mybatis</groupId>                                
        <artifactId>mybatis</artifactId>                              
        <version>3.2.8</version>                                      
    </dependency>                                                           
                                                                                  
    <dependency>                                                            
        <groupId>org.mybatis</groupId>                                
        <artifactId>mybatis-spring</artifactId>                       
        <version>1.2.2</version>                                      
    </dependency>                                                           
                                                                                  
    <!-- 控制日志输出：结合log4j -->                                        
    <dependency>                                                            
        <groupId>log4j</groupId>                                      
        <artifactId>log4j</artifactId>                                
        <version>1.2.17</version>                                     
    </dependency>                                                           
    <dependency>                                                            
        <groupId>org.slf4j</groupId>                                  
        <artifactId>slf4j-api</artifactId>                            
        <version>1.7.7</version>                                      
    </dependency>                                                           
    <dependency>                                                            
        <groupId>org.slf4j</groupId>                                  
        <artifactId>slf4j-log4j12</artifactId>                        
        <version>1.7.7</version>                                      
    </dependency>                                                           
                                                                                  
    <dependency>                                                            
        <groupId>mysql</groupId>                                      
        <artifactId>mysql-connector-java</artifactId>                 
        <version>5.1.37</version>                                     
    </dependency>                                                           
    <dependency>                                                            
        <groupId>jstl</groupId>                                       
        <artifactId>jstl</artifactId>                                 
        <version>1.2</version>                                        
    </dependency>                                                           
                                                                                  
    <!-- ********其他****************************** -->                     
                                                                                  
    <!-- Ehcache二级缓存 -->                                                
    <dependency>                                                            
        <groupId>net.sf.ehcache</groupId>                             
        <artifactId>ehcache</artifactId>                              
        <version>1.6.2</version>                                      
    </dependency>                                                           
                                                                                  
                                                                                  
    <!-- 石英调度 - 开始 -->                                                
    <dependency>                                                            
        <groupId>org.quartz-scheduler</groupId>                       
        <artifactId>quartz</artifactId>                               
        <version>1.8.5</version>                                      
    </dependency>                                                           
    <dependency>                                                            
        <groupId>org.springframework</groupId>                        
        <artifactId>spring-context-support</artifactId>               
        <version>4.0.0.RELEASE</version>                              
    </dependency>                                                           
    <dependency>                                                            
        <groupId>commons-collections</groupId>                        
        <artifactId>commons-collections</artifactId>                  
        <version>3.1</version>                                        
    </dependency>                                                           
    <!-- 石英调度 - 结束 -->                                                
                                                                                  
    <dependency>                                                            
        <groupId>org.codehaus.jackson</groupId>                       
        <artifactId>jackson-mapper-asl</artifactId>                   
        <version>1.9.2</version>                                      
    </dependency>                                                           
                                                                                  
    <dependency>                                                            
        <groupId>org.apache.poi</groupId>                             
        <artifactId>poi</artifactId>                                  
        <version>3.9</version>                                        
    </dependency>                                                           
                                                                                  
    <dependency>                                                            
        <groupId>org.jfree</groupId>                                  
        <artifactId>jfreechart</artifactId>                           
        <version>1.0.19</version>                                     
    </dependency>                                                           
                                                                                  
    <dependency>                                                            
        <groupId>commons-fileupload</groupId>                         
        <artifactId>commons-fileupload</artifactId>                   
        <version>1.3.1</version>                                      
    </dependency>                                                           
                                                                                  
    <dependency>                                                            
        <groupId>org.freemarker</groupId>                             
        <artifactId>freemarker</artifactId>                           
        <version>2.3.19</version>                                     
    </dependency>                                                           
                                                                                  
    <dependency>                                                            
        <groupId>org.activiti</groupId>                               
        <artifactId>activiti-engine</artifactId>                      
        <version>5.15.1</version>                                     
    </dependency>                                                           
                                                                                  
    <dependency>                                                            
        <groupId>org.activiti</groupId>                               
        <artifactId>activiti-spring</artifactId>                      
        <version>5.15.1</version>                                     
    </dependency>                                                           
                                                                                  
    <dependency>                                                            
        <groupId>org.apache.commons</groupId>                         
        <artifactId>commons-email</artifactId>                        
        <version>1.3.1</version>                                      
    </dependency>                                                           
                                                                                  
    <dependency>                                                            
        <groupId>org.activiti</groupId>                               
        <artifactId>activiti-explorer</artifactId>                    
        <version>5.15.1</version>                                     
        <exclusions>                                                        
            <exclusion>                                                     
                <artifactId>groovy-all</artifactId>                   
                <groupId>org.codehaus.groovy</groupId>                
            </exclusion>                                                    
        </exclusions>                                                       
    </dependency>                                                           
</dependencies>                            
```

### 集成Spring框架

在`web.xml`文件中增加配置信息集成`Spring`框架

```xml
  <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:spring/spring-*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
```

`Spring`环境构建时需要读取`web`应用的初始化参数`contextConfigLocation`, 从`classpath`中读取配置文件`spring/spring-*.xml` 因此需要`resources目录中增加`spring`文件夹，并在其中增加`spring-context.xml`配置文件。

`Spring`框架的核心是构建对象，整合对象之间的关系（`IOC`）及扩展对象功能（`AOP`），所以需要在`spring-context.xml`配置文件中增加业务对象扫描的相关配置。扫描后由`Spring`框架进行管理和组合。

```xml
    <context:component-scan base-package="com.hph.*" >
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
```

扫描配置中为什么要排除`Controller`注解Controller注解的的作用是声明控制器（处理器）类。从数据流转的角度，这个类应该是由`SpringMVC`框架进行管理和组织的，所以不需要由`Spring`框架扫描。

### 集成SpringMVC框架

`SpringMVC`框架用于处理系统中数据的流转及控制操作。（从哪里来，到哪里去）

集成`SpringMVC`框架，需要在`web.xml`文件中增加配置信息

```xml
 <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/springmvc-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

`SpringMVC`环境构建时需要读取`servlet`初始化参数`init-param`, 从`classpath`中读取配置文件`spring/springmvc-context.xml`

`SpringMVC`框架的核心是处理数据的流转，所以需要在`springmvc-context.xml`配置文件中增加控制器对象（`Controller`）扫描的相关配置。扫描后由`SpringMVC`框架进行管理和组合。

```xml
    <context:component-scan base-package="com.hph.*" use-default-filters="false" >
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
```

静态资源如何不被`SpringMVC`框架进行拦截:在配置文件中即可

```xml
<mvc:default-servlet-handler/>
<mvc:annotation-driven />
```

在实际的项目中静态资源不会和动态资源放在一起，也就意味着不会放置在服务器中，所以这些配置可以省略。

如果`SpringMVC`框架数据处理为页面跳转，那么需要配置相应的视图解析器`ViewResolver`。

```
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" >
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
```

如果有多个解释器?

`SpringMVC`框架中允许存在多个视图解析器，框架会按照配置声明顺序，依次进行解析。`SpringMVC`框架中配置多个视图解析器时，如果将`InternalResourceViewResolver`解析器配置在前，那么即使找不到视图，框架也不会继续解析，直接发生`404`错误，所以必须将`InternalResourceViewResolver`解析器放置在最后。

如果`SpringMVC`框架数据处理为响应`JSON`字符串，那么为了浏览器方便对响应的字符串进行处理，需要明确字符串的类型及编码方式。



```xml
 <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter" >
        <property name="messageConverters" >
            <list>
                <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter" >
                    <property name="supportedMediaTypes" >
                        <list>
                            <value>application/json;charset=UTF-8</value>
                        </list>
                    </property>
                </bean>
            </list>
        </property>
    </bean>
```

**如果增加了<mvc:annotation-driven />标签，上面面=的配置可省略。**

如果项目中含有文件上传业务，还需要增加文件上传解析器`MultipartResolver`

```xml
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver" p:defaultEncoding="UTF-8" >
        <property name="maxUploadSize" value="2097152"/>
        <property name="resolveLazily" value="true"/>
    </bean>
```

### 集成Mybatis框架

`Mybatis`框架主要处理业务和数据库之间的数据交互，所以创建对象和管理对象生命周期的职责可以委托`Spring`框架完成。如：创建`Mybatis`核心对象。

```xml
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean" >
        <property name="configLocation" value="classpath:mybatis/config.xml" />
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations" >
            <list>
                <value>classpath*:mybatis/mapper-*.xml</value>
            </list>
        </property>
    </bean>
    ...
    <bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer" >
        <property name="basePackage" value="com.atguigu.atcrowdfunding.**.dao" />
    </bean>
```

`Mybatis`核心对象就需要依赖于数据库连接池（`C3P0`）,所以在`Spring`配置文件中增加相应的配置

```xml
   <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" >
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/ssm?rewriteBatchedStatements=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
        <property name="user" value="root"/>
        <property name="password" value="123456"/>
    </bean>
```

集成`Mybatis`框架时同时还需要增加核心配置文件`mybatis/config.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        
    </typeAliases>
</configuration>
```

`SQL`映射文件`mybatis/mapper-SQL.xml`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="xxx.XXDao" >

</mapper>
```

为了保证数据操作的一致性，必须在程序中增加事务处理。`Spring`框架采用声明式事务，通过`AOP`的方式将事务增加到业务中。所以需要在`Spring`配置文件中增加相关配置。

测试前，需要在数据库中增加ssm库及t_user表。

```sql
CREATE DATABASE `ssm`;
CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;
```

## 最终结构图

![AYwDpQ.png](https://s2.ax1x.com/2019/03/24/AYwDpQ.png)

## 最终配置文件

### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:spring/spring-*.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring/springmvc-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
        
```

### srping-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <context:component-scan base-package="com.hph.*">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="configLocation" value="classpath:mybatis/config.xml"/>
        <property name="dataSource" ref="dataSource"/>
        <property name="mapperLocations">
            <list>
                <value>classpath*:mybatis/mapper-*.xml</value>
            </list>
        </property>
    </bean>
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl"
                  value="jdbc:mysql://localhost:3306/ssm?rewriteBatchedStatements=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
        <property name="user" value="root"/>
        <property name="password" value="123456"/>
    </bean>

    <bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.hph.**.dao"/>
    </bean>
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" isolation="DEFAULT" rollback-for="java.lang.Exception"/>
            <tx:method name="query*" read-only="true"/>
        </tx:attributes>
    </tx:advice>
    <aop:config>
        <aop:advisor advice-ref="transactionAdvice" pointcut="execution(* com.hph..*Service.*(..))"/>
    </aop:config>
</beans>
```



### spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

    <context:component-scan base-package="com.hph.*" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <mvc:default-servlet-handler/>
    <mvc:annotation-driven/>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"
          p:defaultEncoding="UTF-8">
        <property name="maxUploadSize" value="2097152"/>
        <property name="resolveLazily" value="true"/>
    </bean>

</beans>
```

### config(mybatis)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>

    </typeAliases>
</configuration>
```

### mapper-SQL.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="xxx.XXDao" >

</mapper>
```

### User

```java
package com.hph.ssm.controller;

import com.hph.ssm.bean.User;
import com.hph.ssm.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/index")
    public String index() {

        return "user/index";
    }

    @ResponseBody
    @RequestMapping("/json")
    public Object json() {
        Map map = new HashMap();
        map.put("username", "张三");
        return map;
    }

    @ResponseBody
    @RequestMapping("/sql")
    public Object sql() {
        List<User> users = userService.queryAll();
        return users;
    }
}
```

### controller

```java
package com.hph.ssm.controller;

import com.hph.ssm.bean.User;
import com.hph.ssm.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/index")
    public String index() {

        return "user/index";
    }

    @ResponseBody
    @RequestMapping("/json")
    public Object json() {
        Map map = new HashMap();
        map.put("username", "张三");
        return map;
    }

    @ResponseBody
    @RequestMapping("/sql")
    public Object sql() {
        List<User> users = userService.queryAll();
        return users;
    }

    @ResponseBody
    @RequestMapping("/aop")
    public Object aop() {
        User user = userService.queryById();
        return user;
    }

}
```

### UserDao

```java
package com.hph.ssm.dao;

import com.hph.ssm.bean.User;
import org.apache.ibatis.annotations.Select;

import java.util.List;

public interface UserDao {
    @Select("select * from t_user where id = 1")
    public User queryById();

    @Select("select * from t_user  ")
    List<User>  queryAll();
}
```

### UserService

```java
package com.hph.ssm.service;

import com.hph.ssm.bean.User;

import java.util.List;

public interface UserService {
    public User queryById();

    List<User> queryAll();
}
```

### UserServiceImpl

```java
package com.hph.ssm.service.impl;

import com.hph.ssm.bean.User;
import com.hph.ssm.dao.UserDao;
import com.hph.ssm.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDao userDao;

    public User queryById() {
        return userDao.queryById();
    }

    public List<User> queryAll() {
        return userDao.queryAll();
    }

}
```

![AYwoc9.png](https://s2.ax1x.com/2019/03/24/AYwoc9.png)

配置Tomcat将项目发送服务器

## 测试

### SpringMVC

如果访问成功，说明项目中`SpringMVC`框架集成成功。

![AYwb0x.png](https://s2.ax1x.com/2019/03/24/AYwb0x.png)

SpringMVC框架集成成功

### Spring框架(IOC功能)

![AYwXtO.png](https://s2.ax1x.com/2019/03/24/AYwXtO.png)

Spring框架（`IOC`功能）集成成功。

### Spring框架(AOP功能)

![AY0e3Q.png](https://s2.ax1x.com/2019/03/24/AY0e3Q.png)

`Spring`框架（`AOP`功能）集成成功

### Mybatis

![AY084U.png](https://s2.ax1x.com/2019/03/24/AY084U.png)

![AY03NT.png](https://s2.ax1x.com/2019/03/24/AY03NT.png)



## 模拟生产环境

Web软件开发中，由于开发阶段不同，系统环境主要分为：开发环境，测试环境，生产环境。

将系统部署到生产环境后，经常会听开发人员这么说：这个`Bug`不应该呀，我们在测试时没问题呀，怎么到了生产环境就不行了呢!!!

其实之所以会出现这种情况，很大程度上是因为我们在项目的不同阶段时，对应用系统的访问方式不一样所造成的。

一般在开发，测试阶段时，我们都会以本地服务器地址`http://localhost:8080/project`访问系统，但是在生产环境中，我们的访问方式发生了变化：`http://com.xxxxx/`，由于环境的不同，导致访问方式的变化，那么就会产生很多之前没有的问题。

**如果能将开发，测试，生产环境的访问方式保持一致的话，那么就可以提前发现客户在使用时所发生的问题，将这些问题提前解决，还是非常不错的。**

- 将`Tomcat`服务器的默认`HTTP`监听端口号：`8080`修改为`80`
- 将项目中`.settings`目录下的配置文件`org.eclipse.wst.common.component`中的`context-root`属性修改为`/(斜杠)`(Eclipse)
- IDEA直接在Tomcat服务器中更改Deployment中的Application context为 /
- 修改系统主机文件`c:\Windows\System32\drivers\etc\hosts`增加`IP`地址和域名的解析关系：`127.0.0.1 www.hph.com`

![AY068e.png](https://s2.ax1x.com/2019/03/24/AY068e.png)

## 参考资料

尚硅谷SSM教学视频













