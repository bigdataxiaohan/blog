---
title: JdbcTemplate
date: 2019-03-05 12:26:20
tags: Spring
categories: JavaWeb
---

### 概述

​         为了使JDBC更加易于使用，Spring在JDBC API上定义了一个抽象层，以此建立一个JDBC存取框架。 作为Spring JDBC框架的核心，JDBC模板的设计目的是为不同类型的JDBC操作提供模板方法，通过这种方式，可以在尽可能保留灵活性的情况下，将数据库存取的工作量降到最低。 可以将Spring的JdbcTemplate看作是一个小型的轻量级持久化层框架，和我们之前使用过的DBUtils风格非常接近。

### jdbc.properties

```properties
jdbc.username=root	//此处请带前缀否则会导致${username}为系统用户名
jdbc.password=123456
jdbc.url=jdbc:mysql://localhost:3306/bigdata
jdbc.driver=com.mysql.jdbc.Driver
initialPoolSize=30
minPoolSize=10
maxPoolSize=100
acquireIncrement=5
maxStatements=1000
maxStatementsPerConnection=10
```

### spring配置文件

```xml
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"></property>
        <property name="jdbcUrl" value="${jdbc.url}"></property>
        <property name="user" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
        <property name="initialPoolSize" value="${initialPoolSize}"/>
        <property name="minPoolSize" value="${minPoolSize}"/>
        <property name="maxPoolSize" value="${maxPoolSize}"/>
        <property name="acquireIncrement" value="${acquireIncrement}"/>
        <property name="maxStatements" value="${maxStatements}"/>
        <property name="maxStatementsPerConnection"
                  value="${maxStatementsPerConnection}"/>
    </bean>
    <!--JdbcTemplate-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```

### 持久化操作

**增删改: **JdbcTemplate.update(String, Object...)

**批量增删改: **JdbcTemplate.batchUpdate(String, List<Object[]>) 

Object[]封装了SQL语句每一次执行时所需要的参数

List集合封装了SQL语句多次执行时的所有参数

**查询单行 **JdbcTemplate.queryForObject(String, RowMapper&lt;Department&gt;, Object...)

**查询多行:**JdbcTemplate.query(String, RowMapper&lt;Department&gt;, Object...)  RowMapper对象依然可以使用BeanPropertyRowMapper

**查询单一值:**JdbcTemplate.queryForObject(String, Class, Object...)

### 前置准备

```java

public class Employee {
	
	private Integer id ; 
	private String lastName; 
	private String email ;
	private Integer gender;
	
	public Employee() {
		// TODO Auto-generated constructor stub
	}
	
	public Employee(Integer id, String lastName, String email, Integer gender) {
		super();
		this.id = id;
		this.lastName = lastName;
		this.email = email;
		this.gender = gender;
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
	@Override
	public String toString() {
		return "Employee [id=" + id + ", lastName=" + lastName + ", email=" + email + ", gender=" + gender + "]";
	} 
	
}
```

### 方法实现

```java
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.jdbc.core.namedparam.SqlParameterSource;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class TestJdbc {

    private JdbcTemplate jdbcTemplate;

    private NamedParameterJdbcTemplate npjt;
	//前置操作初始化
    @Before
    public void init() {
        ApplicationContext ctx =
                new ClassPathXmlApplicationContext("spring-jdbc.xml");

        jdbcTemplate = ctx.getBean("jdbcTemplate", JdbcTemplate.class);

        npjt = ctx.getBean("namedParameterJdbcTemplate", NamedParameterJdbcTemplate.class);

    }

    /**
     * update():  增删改操作
     */
    @Test
    public void testUpdate() {
        String sql = "insert into tbl_employee(last_name,email,gender) value(?,?,?)";

        //jdbcTemplate.update(sql, "运慧","yh@atguigu.com",1);
        jdbcTemplate.update(sql, new Object[]{"QFX", "QFX@atguigu.com", 1});
    }

    /**
     * batchUpdate(): 批量增删改
     * 作业: 批量删  修改
     */
    @Test
    public void testBatchUpdate() {
        String sql = "insert into tbl_employee(last_name,email,gender) value(?,?,?)";
        List<Object[]> batchArgs = new ArrayList<Object[]>();
        batchArgs.add(new Object[]{"zsf", "zsf@sina.com", 1});
        batchArgs.add(new Object[]{"zwj", "zwj@sina.com", 1});
        batchArgs.add(new Object[]{"sqs", "sqs@sina.com", 1});

        jdbcTemplate.batchUpdate(sql, batchArgs);
    }


    /**
     * queryForObject():
     * 1. 查询单行数据 返回一个对象
     * 2. 查询单值 返回单个值
     */
    @Test
    public void testQueryForObjectReturnObject() {
        String sql = "select id,last_name,email,gender from tbl_employee where id = ?";
        //rowMapper: 行映射  将结果集的一条数据映射成具体的一个java对象.
        RowMapper<Employee> rowMapper = new BeanPropertyRowMapper<>(Employee.class);

        Employee employee = jdbcTemplate.queryForObject(sql, rowMapper, 1001);
        System.out.println(employee);
    }

    @Test
    public void testQueryForObjectReturnValue() {
        String sql = "select count(id) from tbl_employee";

        Integer result = jdbcTemplate.queryForObject(sql, Integer.class);
        System.out.println(result);
    }

    /**
     * query(): 查询多条数据返回多个对象的集合.
     */

    @Test
    public void testQuery() {
        String sql = "select id,last_name,email,gender from tbl_employee";
        RowMapper<Employee> rowMapper = new BeanPropertyRowMapper<>(Employee.class);

        List<Employee> emps = jdbcTemplate.query(sql, rowMapper);
        System.out.println(emps);
    }


    /**
     * 测试具名参数模板类
     */

    @Test
    public void testNpjt() {
        String sql = "insert into tbl_employee(last_name,email,gender) values(:ln,:em,:ge)";
        Map<String, Object> paramMap = new HashMap<>();

        paramMap.put("ln", "Jerry");
        paramMap.put("em", "jerry@sina.com");
        paramMap.put("ge", 0);


        npjt.update(sql, paramMap);
    }


    @Test
    public void testNpjtObject() {
        //模拟Service层 直接传递给Dao层一个具体的  对象
        Employee employee = new Employee(null, "张无忌", "zwj@sina.com", 1);

        //在dao的插入方法中:
        String sql = "insert into tbl_employee(last_name,email,gender) values(:lastName,:email,:gender)";

        SqlParameterSource paramSource = new BeanPropertySqlParameterSource(employee);

        npjt.update(sql, paramSource);

    }


}
```









