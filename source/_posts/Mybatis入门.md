---
title: Mybatis入门
date: 2019-03-16 15:13:56
tags: Mybatis
categories: JavaWeb
---

## 简介

MyBatis是Apache的一个开源项目iBatis, 2010年6月这个项目由Apache Software      Foundation 迁移到了Google Code，随着开发团队转投Google Code旗下， iBatis3.x   正式更名为MyBatis ，代码于2013年11月迁移到Github

iBatis一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。 iBatis  提供的持久层框架包括SQL Maps和Data Access Objects（DAO）

## 特点

- MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架
- MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集
- MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录
- 其是一个半自动ORM（Object Relation Mapping对象关系映射）框架   Hibernant是全自动的

## 案例

### 结构

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.hph</groupId>
    <artifactId>Mybatis</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <!--mybatis版本-->
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.1</version>
        </dependency>
        <dependency>
            <!--log4j版本-->
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.25</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <!--mysql驱动-->
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.37</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</project>
```

#### Employee

```java
package com.hph.mybatis.beans;

public class Employee {
    private Integer id;
    private String lastName;
    private String email;
    private Integer gender;


    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", lastName='" + lastName + '\'' +
                ", email='" + email + '\'' +
                ", gender=" + gender +
                '}';
    }

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

    public Integer getGender() {
        return gender;
    }

    public void setGender(Integer gender) {
        this.gender = gender;
    }
}

```

#### EmployeeDao

```java
package com.hph.mybatis.dao;

import com.hph.mybatis.beans.Employee;

public interface EmployeeDao {
    public Employee getEmployeeById(Integer id);
}
```

#### TestMybatis

```java
package com.hph.mybatis.test;

import com.hph.mybatis.beans.Employee;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;

public class TestMybatis {
    @Test
    public void testSqlsessionFactory() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        System.out.println(sqlSessionFactory);
        SqlSession session = sqlSessionFactory.openSession();
        System.out.println(session);
        try {
            Employee employee = session.selectOne("suibian.selectEmployee", 1001);
            System.out.println(employee);
        } finally {
            session.close();
        }
    }
```

#### db.protertis

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://58.87.70.124:3306/test_mybatis
jdbc.username=root
jdbc.password=123456
```

#### EmployeeMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE mapper  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--配置SQL映射-->
<mapper namespace="suibian">
    <select id="selectEmployee" resultType="com.hph.mybatis.beans.Employee">
    select * from tbl_employee where id = #{id}  </select>
</mapper>
```

#### mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 配置 -->
<configuration>
    <properties resource="db.properties">
    </properties>
    <settings>
        <!-- 映射下划线到驼峰命名 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <typeAliases>
        <!--  <typeAlias type="com.hph.mybatis.beans.Employee" alias="employee"/> -->
        <package name="com.hph.mybatis.beans"/>
    </typeAliases>
    <environments default="development">
        <!-- 具体的环境 -->
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="EmployeeMapper.xml"></mapper>
    </mappers>
</configuration>
```

![AVj2qA.png](https://s2.ax1x.com/2019/03/16/AVj2qA.png)









