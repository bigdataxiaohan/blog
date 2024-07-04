---
title: Mybatis之动态SQL
date: 2019-03-19 12:36:05
tags: Mybatis
categories: JavaWeb
---

## 简介

- 动态SQL是MyBatis强大特性之一。极大的简化我们拼装SQL的操作
- 动态SQL元素和使用 JSTL 或其他类似基于 XML 的文本处理器相似
- MyBatis 采用功能强大的基于 OGNL 的表达式来简化操作
    - If
    - choose (when, otherwise)
    - trim (where, set)
    - foreach

OGNL（ Object Graph Navigation Language ）对象图导航语言，这是一种强大的表达式语言，通过它可以非常方便的来操作对象属性。 类似于我们的EL，SpEL等

| 功能             | 参数                                             |
| ---------------- | ------------------------------------------------ |
| 访问对象属性     | person.name                                      |
| 调用方法         | person.getName()                                 |
| person.getName() | @java.lang.Math@PI  @java.util.UUID@randomUUID() |
| 调用构造方法     | new<br/>com.atguigu.bean.Person(‘admin’).name    |
| 运算符           | +,-*,/,%                                         |
| 逻辑运算符       | in,not in,>,>=,<,<=,==,!=                        |

**注意：xml中特殊符号如”,>,<等这些都需要使用转义字符**

## 项目结构

## 数据准备

![Amx8Z4.png](https://s2.ax1x.com/2019/03/18/Amx8Z4.png)

![AmxZIs.png](https://s2.ax1x.com/2019/03/18/AmxZIs.png)

## 项目结构

![Amxyod.png](https://s2.ax1x.com/2019/03/18/Amxyod.png)

### Department

```java
package com.hph.mybatis.beans;

import java.util.List;

public class Department {

    private Integer id;
    private String departmentName ;

    private List<Employee> emps ;


    public List<Employee> getEmps() {
        return emps;
    }
    public void setEmps(List<Employee> emps) {
        this.emps = emps;
    }
    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }
    public String getDepartmentName() {
        return departmentName;
    }
    public void setDepartmentName(String departmentName) {
        this.departmentName = departmentName;
    }
    @Override
    public String toString() {
        return "Department [id=" + id + ", departmentName=" + departmentName + "]";
    }
}
```

### Employee

```java
package com.hph.mybatis.beans;

public class Employee {

    private Integer id;
    private String lastName;
    private String email;
    private Integer gender;

    private Department dept;

    public void setDept(Department dept) {
        this.dept = dept;
    }

    public Department getDept() {
        return dept;
    }

    public Employee() {
    }

    public Employee(Integer id, String lastName, String email, Integer gender, Department dept) {
        this.id = id;
        this.lastName = lastName;
        this.email = email;
        this.gender = gender;
        this.dept = dept;
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

### EmployeeMapperDynamicSQL

```java
package com.hph.mybatis.dao;

import com.hph.mybatis.beans.Employee;
import org.apache.ibatis.annotations.Param;

import java.util.List;

public interface EmployeeMapperDynamicSQL {

    public List<Employee> getEmpsByConditionIfWhere(Employee condition);

    public List<Employee> getEmpsByConditionTrim(Employee condition);

    public void updateEmpByConitionSet(Employee condition);

    public  List<Employee> getEmpsByConditionChoose(Employee condition);

    public List<Employee> getEmpsByIds(@Param("ids") List<Integer> ids);

    //批量操作: 删除 修改 添加
    public void addEmps(@Param("emps") List<Employee> emps);
}
```

### EmployeeMapperDynamicSQL.xml

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.hph.mybatis.dao.EmployeeMapperDynamicSQL">
    <!-- public List<Employee>  getEmpsByConditionIfWhere(Employee Condition); -->
    <select id="getEmpsByConditionIfWhere" resultType="com.hph.mybatis.beans.Employee">
        select id, last_name, email, gender
        from tbl_employee
        <!-- where 1=1 -->
        <where> <!-- 在SQL语句中提供WHERE关键字，  并且要解决第一个出现的and 或者是 or的问题 -->
            <if test="id!=null">
                and id = #{id }
            </if>
            <if test="lastName!=null&amp;&amp;lastName!=&quot;&quot;">
                and last_name = #{lastName}
            </if>
            <if test="email!=null and email.trim()!=''">
                and email = #{email}
            </if>
            <if test="gender==0 or gender==1">
                and gender = #{gender}
            </if>
        </where>
    </select>
    <select id="getEmpsByConditionTrim" resultType="com.hph.mybatis.beans.Employee">
        select id, last_name, email, gender
        from tbl_employee
        <!--
        prefix: 添加一个前缀
        pprefixOverrides:覆盖/去掉一个前缀
        ssuffxi:添加一个后缀
        suffixOverrides:覆盖/去掉一个后缀
         -->

        <trim prefix="where" suffixOverrides="and|or">
            <if test="id!=null">
                id = #{id } and
            </if>
            <if test="lastName!=null&amp;&amp;lastName!=&quot;&quot;">
                last_name = #{lastName} and
            </if>
            <if test="email!=null and email.trim()!=''">
                email = #{email} or
            </if>
            <if test="gender==0 or gender==1">
                gender = #{gender}
            </if>
        </trim>
    </select>
    <!--public void updateEmpByConitionSet(Employee condition);-->

    <update id="updateEmpByConitionSet">
        update tbl_employee
        <set>
            <if test="lastName!=null">
                last_name = #{lastName},
            </if>
            <if test="email!=null">
                email = #{email},
            </if>
            <if test="gender==0 or gender==1">
                gender= #{gender},
            </if>
        </set>
        where id = #{id}
    </update>
    <!--    public void updateEmpByConitionSet(Employee condition);-->
    <select id="getEmpsByConditionChoose" resultType="com.hph.mybatis.beans.Employee">
        select id,last_name,email,gender
        from tbl_employee
        where
        <choose>
            <when test="id!=null">
                id=#{id}
            </when>
            <when test="lastName!=null">
                last_name=#{lastName}
            </when>
            <when test="email!=null">
                email = #{email}
            </when>
            <otherwise>
                gender = 0
            </otherwise>
        </choose>
    </select>

    <!--    public List<Employee> getEmpsByIds(@Param("ids")List<Integer> ids);-->
    <select id="getEmpsByIds" resultType="com.hph.mybatis.beans.Employee">
        <!--
        foreach:
            collection:指定要迭代的几乎额
            item:当前集合中迭代出的元素
            open:指定一个开始字符
            close:指定一个结束字符
            separtor:元素与元素之间的分隔符
        -->
        select id,last_name,email,gender from tbl_employee
        where id in
        <foreach collection="ids" item="currId" open="(" close=")" separator=",">
            #{currId}
        </foreach>
    </select>

    <!--    public void addEmps(@Param("emps") List<Employee> emps)
    添加:insert into tbl_employee(x,x,x) values(?,?,?),(?,?<?),(?,?,?)
    删除:delete from tbl_employee where id in (?,?,?)
    修改:update tbl_employee set last_name = #{lastName }...where id  = #{id}}
         update tbl_employee set last_name = #{lastName }...where id  = #{id}}
         update tbl_employee set last_name = #{lastName }...where id  = #{id}}
         默认情况下,JDBCb允许将多条分号SQL通过;平日你改成一个字符串

    ;-->

    <insert id="addEmps">
        insert into tbl_employee(last_name, email,gender ) values
        <foreach collection="emps" item="emp" separator=",">
            (#{emp.lastName},#{emp.email},#{emp.gender})
        </foreach>
    </insert>

</mapper>
```



### TestMybatisDynamicSQL

```java
package com.hph.mybatis.test;

import com.hph.mybatis.beans.Employee;
import com.hph.mybatis.dao.EmployeeMapperDynamicSQL;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

public class TestMybatisDynamicSQL {

    public SqlSessionFactory getSqlSessionFactory() throws Exception {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        return sqlSessionFactory;
    }

    @Test
    public void testIf() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession();

        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
         /*   condition.setId(5);
            condition.setLastName("清风_笑丶");*/
            //condition.setEmail("qfx@sina.com");
            //condition.setGender(1);
            List<Employee> emps = mapper.getEmpsByConditionIfWhere(condition);
            System.out.println(emps);

        } finally {
            session.close();
        }
    }

    @Test
    public void testTrim() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession();

        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
            condition.setId(5);
            condition.setLastName("清风_笑丶");
            condition.setEmail("qfx@gmail.com");
            //condition.setGender(1);
            List<Employee> emps = mapper.getEmpsByConditionTrim(condition);
            System.out.println(emps);

        } finally {
            session.close();
        }
    }

    @Test
    public void testSet() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession(true);

        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
            condition.setId(5);
            condition.setLastName("清风_笑丶testSet");
            condition.setEmail("qf_x@qq.com");
//         condition.setGender(1);
            mapper.updateEmpByConitionSet(condition);
        } finally {
            session.close();
        }
    }

    @Test
    public void testChoose() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession(true);

        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
            condition.setId(1);
            condition.setLastName("清风_笑丶testChoose");
            condition.setEmail("qf_xtestChoose@gmail.com");
            //condition.setGender(1);
            List<Employee> emps = mapper.getEmpsByConditionChoose(condition);
            System.out.println(emps);
        } finally {
            session.close();
        }
    }

    @Test
    public void testForeach() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession(true);
        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
            List<Integer> ids = new ArrayList<Integer>();
            ids.add(5);
            ids.add(6);
            ids.add(7);
            List<Employee> emps = mapper.getEmpsByIds(ids);

            System.out.println(emps);
        } finally {
            session.close();
        }
    }

    @Test
    public void testBatch() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession(true);
        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            List<Employee> emps = new ArrayList<Employee>();
            emps.add(new Employee(null, "清风笑_testBatch1", "qfx_testBatch1@sina.com", 1));
            emps.add(new Employee(null, "清风笑_testBatch2", "qfx_testBatch2@sina.com", 0));
            emps.add(new Employee(null, "清风笑_testBatch3", "qfx_testBatch3@sina.com", 1));

            mapper.addEmps(emps);

        } finally {
            session.close();
        }
    }
}
```

## if  where

 If用于完成简单的判断.

Where用于解决SQL语句中where关键字以及条件中第一个and或者or的问题 

```xml
  <!-- public List<Employee>  getEmpsByConditionIfWhere(Employee Condition); -->
    <select id="getEmpsByConditionIfWhere" resultType="com.hph.mybatis.beans.Employee">
        select id, last_name, email, gender
        from tbl_employee
        <!-- where 1=1 -->
        <where> <!-- 在SQL语句中提供WHERE关键字，  并且要解决第一个出现的and 或者是 or的问题 -->
            <if test="id!=null">
                and id = #{id }
            </if>
            <if test="lastName!=null&amp;&amp;lastName!=&quot;&quot;">
                and last_name = #{lastName}
            </if>
            <if test="email!=null and email.trim()!=''">
                and email = #{email}
            </if>
            <if test="gender==0 or gender==1">
                and gender = #{gender}
            </if>
        </where>
    </select>
```

```java
   public void testIf() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession();

        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
            List<Employee> emps = mapper.getEmpsByConditionIfWhere(condition);
            System.out.println(emps);
        } finally {
            session.close();
        }
    }
```

![Anokv9.png](https://s2.ax1x.com/2019/03/19/Anokv9.png)

```java
   @Test
    public void testIf() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession();

        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
            condition.setId(1001);
            List<Employee> emps = mapper.getEmpsByConditionIfWhere(condition);
            System.out.println(emps);

        } finally {
            session.close();
        }
    }
```

![AnotVP.png](https://s2.ax1x.com/2019/03/19/AnotVP.png)

## trim

```xml
    <select id="getEmpsByConditionTrim" resultType="com.hph.mybatis.beans.Employee">
        select id, last_name, email, gender
        from tbl_employee
        <!--
        prefix: 添加一个前缀
        pprefixOverrides:覆盖/去掉一个前缀
        ssuffxi:添加一个后缀
        suffixOverrides:覆盖/去掉一个后缀
         -->

        <trim prefix="where" suffixOverrides="and|or">
            <if test="id!=null">
                id = #{id } and
            </if>
            <if test="lastName!=null&amp;&amp;lastName!=&quot;&quot;">
                last_name = #{lastName} and
            </if>
            <if test="email!=null and email.trim()!=''">
                email = #{email} or
            </if>
            <if test="gender==0 or gender==1">
                gender = #{gender}
            </if>
        </trim>
    </select>
```



```java
  @Test
    public void testTrim() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession();

        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
            List<Employee> emps = mapper.getEmpsByConditionTrim(condition);
            System.out.println(emps);

        } finally {
            session.close();
        }
    }
```

![AnocV0.png](https://s2.ax1x.com/2019/03/19/AnocV0.png)

##  set 

set 主要是用于解决修改操作中SQL语句中可能多出逗号的问题.

```xml
<update id="updateEmpByConitionSet">
    update tbl_employee
    <set>
        <if test="lastName!=null">
            last_name = #{lastName},
        </if>
        <if test="email!=null">
            email = #{email},
        </if>
        <if test="gender==0 or gender==1">
            gender= #{gender},
        </if>
    </set>
    where id = #{id}
</update>
```



```java
    @Test
    public void testSet() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession(true);

        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
            condition.setId(1001);
            condition.setLastName("清风笑丶");
            condition.setEmail("qf_x@gmail.com");
            mapper.updateEmpByConitionSet(condition);
        } finally {
            session.close();
        }
    }
```

![AnTKZq.png](https://s2.ax1x.com/2019/03/19/AnTKZq.png)

![AnTQoV.png](https://s2.ax1x.com/2019/03/19/AnTQoV.png)


##  choose(when、otherwise)


choose 主要是用于分支判断，类似于java中的switch case,只会满足所有分支中的一个

```xml
<!--    public void updateEmpByConitionSet(Employee condition);-->
<select id="getEmpsByConditionChoose" resultType="com.hph.mybatis.beans.Employee">
    select id,last_name,email,gender
    from tbl_employee
    where
    <choose>
        <when test="id!=null">
            id=#{id}
        </when>
        <when test="lastName!=null">
            last_name=#{lastName}
        </when>
        <when test="email!=null">
            email = #{email}
        </when>
        <otherwise>
            gender = 0
        </otherwise>
    </choose>
</select>
```

```java
 @Test
    public void testChoose() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession(true);

        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
            
            List<Employee> emps = mapper.getEmpsByConditionChoose(condition);
            System.out.println(emps);
        } finally {
            session.close();
        }
    }
```

![An7b9O.png](https://s2.ax1x.com/2019/03/19/An7b9O.png)

## foreach

foreach 主要用户循环迭代

&nbsp;&nbsp;&nbsp;&nbsp;collection: 要迭代的集合

&nbsp;&nbsp;&nbsp;&nbsp;item: 当前从集合中迭代出的元素

&nbsp;&nbsp;&nbsp;&nbsp;open: 开始字符

&nbsp;&nbsp;&nbsp;&nbsp;close:结束字符

&nbsp;&nbsp;&nbsp;&nbsp;separator: 元素与元素之间的分隔符

&nbsp;&nbsp;&nbsp;&nbsp;index:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;迭代的是List集合: index表示的当前元素的下标

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;迭代的Map集合:  index表示的当前元素的key

```xml
    <!--    public List<Employee> getEmpsByIds(@Param("ids")List<Integer> ids);-->
    <select id="getEmpsByIds" resultType="com.hph.mybatis.beans.Employee">
        <!--
        foreach:
            collection:指定要迭代的几乎额
            item:当前集合中迭代出的元素
            open:指定一个开始字符
            close:指定一个结束字符
            separtor:元素与元素之间的分隔符
        -->
        select id,last_name,email,gender from tbl_employee  where id in
        <foreach collection="ids" item="currId" open="(" close=")" separator=",">
            #{currId}
        </foreach>
    </select>
```

```java
    @Test
    public void testForeach() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession(true);
        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            Employee condition = new Employee();
            List<Integer> ids = new ArrayList<Integer>();
            ids.add(1001);
            ids.add(1002);
            List<Employee> emps = mapper.getEmpsByIds(ids);

            System.out.println(emps);
        } finally {
            session.close();
        }
    }
```

![AnHorj.png](https://s2.ax1x.com/2019/03/19/AnHorj.png)

```xml
<!--
    //批量操作: 删除 修改 添加
    public void addEmps(@Param("emps") List<Employee> emps);-->
<insert id="addEmps">
    insert into tbl_employee(last_name, email,gender ) values
    <foreach collection="emps" item="emp" separator=",">
        (#{emp.lastName},#{emp.email},#{emp.gender})
    </foreach>
</insert>
```

```java
  @Test
    public void testBatch() throws Exception {
        SqlSessionFactory ssf = getSqlSessionFactory();
        SqlSession session = ssf.openSession(true);
        try {
            EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
            List<Employee> emps = new ArrayList<Employee>();
            emps.add(new Employee(null, "清风笑_testBatch1", "qfx_1@sina.com", 1));
            emps.add(new Employee(null, "清风笑_testBatch2", "qfx_2@sina.com", 0));
            emps.add(new Employee(null, "清风笑_testBatch3", "qfx_3@sina.com", 1));

            mapper.addEmps(emps);

        } finally {
            session.close();
        }
    }
```

![AnbnLd.png](https://s2.ax1x.com/2019/03/19/AnbnLd.png)

## sql

 sql 标签是用于抽取可重用的sql片段，将相同的，使用频繁的SQL片段抽取出来，单独定义，方便多次引用.

抽取SQL

```xml
<sql id="selectSQL">
		select id , last_name, email ,gender from tbl_employee
</sql> 

```

引用SQL:

```xml
<include refid="selectSQL"></include>
```

