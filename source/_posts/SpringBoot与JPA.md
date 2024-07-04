---
title: SpringBoot与JPA
date: 2019-04-09 10:06:02
tags: JPA
categories: SpringBoot 
---

## 简介

JPA是Java Persistence API的简称，中文名Java持久层API，是JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。

![A5GTm9.png](https://s2.ax1x.com/2019/04/08/A5GTm9.png)

## 准备

![A5qYx1.png](https://s2.ax1x.com/2019/04/09/A5qYx1.png)

![A5qLLV.png](https://s2.ax1x.com/2019/04/09/A5qLLV.png)

### Maven

Maven的依赖关系

![A5qjdU.png](https://s2.ax1x.com/2019/04/09/A5qjdU.png)

### 目录结构

![A5LlOP.png](https://s2.ax1x.com/2019/04/09/A5LlOP.png)

```java
package com.hph.springboot.repository;

import com.hph.springboot.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

//继承JpaRepository来完成对数据库的操作
public interface UserRepository extends JpaRepository<User,Integer> {
}
```

```java
package com.hph.springboot.entity;


import javax.persistence.*;

//使用JPA注解配置映射关系
@Entity //告诉JPA这是一个实体类（和数据表映射的类）
@Table(name = "tbl_user") //@Table来指定和哪个数据表对应;如果省略默认表名就是user；
public class User {

    @Id //这是一个主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增主键
    private Integer id;

    @Column(name = "last_name",length = 50) //这是和数据表对应的一个列
    private String lastName;
    @Column //省略默认列名就是属性名
    private String email;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

```java
package com.hph.springboot.controller;

import com.hph.springboot.entity.User;
import com.hph.springboot.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @Autowired
    UserRepository userRepository;

    @GetMapping("/user/{id}")
    public User getUser(@PathVariable("id") Integer id){
        User user = userRepository.findOne(id);
        return user;
    }

    @GetMapping("/user")
    public User insertUser(User user){
        User save = userRepository.save(user);
        return save;
    }
}
```


```yaml
spring:
  datasource:
    url: jdbc:mysql://192.168.1.110/jpa
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    hibernate:
      #     更新或者创建数据表结构
      ddl-auto: update
    #    控制台显示SQL
    show-sql: true
```

## 运行

![A5L5m6.png](https://s2.ax1x.com/2019/04/09/A5L5m6.png)

创建tbl_user表。

![A5O9AS.png](https://s2.ax1x.com/2019/04/09/A5O9AS.png)

## 测试 

### 数据准备

![A5Od4e.png](https://s2.ax1x.com/2019/04/09/A5Od4e.png)

### 查询

![A5OrjI.png](https://s2.ax1x.com/2019/04/09/A5OrjI.png)

