---
title: Mybatis的resultMap自定义映射
date: 2019-03-19 10:31:16
tags: Mybatis
categories: JavaWeb
---

## 自定义映射

- 自定义resultMap，实现高级结果集映射
- id ：用于完成主键值的映射
- result ：用于完成普通列的映射
- association ：一个复杂的类型关联;许多结果将包成这种类型
- collection ： 复杂类型的集

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

### EmployeeMapperResultMap

```java
package com.hph.mybatis.dao;

import com.hph.mybatis.beans.Employee;

import java.util.List;

public interface EmployeeMapperResultMap {

    public Employee getEmployeeById(Integer id);

    public Employee getEmpAndDept(Integer id);

    public  Employee getEmpAndDeptStep(Integer id);

    public List<Employee> getEmpsByDid(Integer did);

}

```

### EmployeeMapperResultMap.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.hph.mybatis.dao.EmployeeMapperResultMap">
    <!-- 自定义映射
        type: 最终结果集封装的类型
        <id>: 完成主键列的映射
            column: 指定结果集的列名
            property:指定对象的属性名
        <result>:完成普通列的映射
     -->
    <select id="getEmployeeById" resultMap="MyEmp">
		select id ,last_name, email,gender from tbl_employee where id = #{id}
	</select>
    <resultMap type="com.hph.mybatis.beans.Employee" id="MyEmp">
        <id column="id" property="id"/>
        <result column="last_name" property="lastName"/>
        <result column="email" property="email"/>
        <result column="gender" property="gender"/>
    </resultMap>

    <!--
		需求: 查询员工对象， 并且查询员工所在 的部门信息.
	 -->
    <!-- public Employee getEmpAndDept(Integer id ); -->
    <select id="getEmpAndDept" resultMap="myEmpAndDept">
        SELECT e.id eid ,  e.last_name, e.email,e.gender  , d.id did , d.dept_name
	    FROM  tbl_employee  e  , tbl_dept  d
	    WHERE e.d_id = d.id  AND e.id = #{id}
    </select>
    <resultMap type="com.hph.mybatis.beans.Employee" id="myEmpAndDept">
        <id column="eid" property="id"/>
        <result column="last_name" property="lastName"/>
        <result column="email" property="email"/>
        <result column="gender" property="gender"/>
        <!-- 级联的方式 -->
        <result column="did" property="dept.id"/>
        <result column="dept_name" property="dept.departmentName"/>
    </resultMap>
    <!--
        association 使用分步查询:
        需求:  查询员工信息并且查询员工所在的部门信息.
              1. 先根据员工的id查询员工信息
              2. 使用外键 d_id查询部门信息
     -->
    <!--public  Employee getEmpAndDeptStep(Integer id);-->
    <select id="getEmpAndDeptStep" resultMap="myEmpAndDeptStep">
	  	select id, last_name, email,gender ,d_id  from tbl_employee where id = #{id}
	  </select>
    <resultMap type="com.hph.mybatis.beans.Employee" id="myEmpAndDeptStep">
        <id column="id" property="id"/>
        <result column="last_name" property="lastName"/>
        <result column="email" property="email"/>
        <result column="gender" property="gender"/>
        <!-- 分步查询 -->
        <association property="dept"
                     select="com.hph.mybatis.dao.DepartmentMapperResultMap.getDeptById"
                     column="{did=d_id}" fetchType="eager">
        </association>
    </resultMap>
    <!-- association 分步查询使用延迟加载/懒加载:
            在全局配置文件中加上两个settings设置:
            <setting name="lazyLoadingEnabled" value="true"/>
          <setting name="aggressiveLazyLoading" value="false"/>
     -->
    <!-- public List<Employee>  getEmpsByDid(Integer did ); -->
    <select id="getEmpsByDid" resultType="com.hph.mybatis.beans.Employee">
        select  id,last_name,email,gender from  tbl_employee where  d_id = #{did}
    </select>
</mapper>
```

### TestMapbatisResultMap

```java
package com.hph.mybatis.test;

import com.hph.mybatis.beans.Department;
import com.hph.mybatis.beans.Employee;
import com.hph.mybatis.dao.DepartmentMapperResultMap;
import com.hph.mybatis.dao.EmployeeMapperResultMap;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;

public class TestMapbatisResultMap {

    public SqlSessionFactory getsqlSessionFactory() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        return sqlSessionFactory;
    }

    @Test
    public void testResultMap() throws IOException {
        SqlSessionFactory ssf = getsqlSessionFactory();
        SqlSession session = ssf.openSession();
        try {
            EmployeeMapperResultMap mapper = session.getMapper(EmployeeMapperResultMap.class);
            Employee employee = mapper.getEmployeeById(1001);
            System.out.println(employee);
        } finally {
            session.close();
        }
    }

    @Test
    public void testResultMapCascade() throws IOException {
        SqlSessionFactory ssf = getsqlSessionFactory();
        SqlSession session = ssf.openSession();
        try {
            EmployeeMapperResultMap mapper = session.getMapper(EmployeeMapperResultMap.class);

            Employee employee = mapper.getEmpAndDept(1001);
            System.out.println(employee);
            System.out.println(employee.getDept());

        } finally {
            session.close();
        }
    }

    @Test
    public void testResultMapAssociation() throws IOException {
        SqlSessionFactory ssf = getsqlSessionFactory();
        SqlSession session = ssf.openSession();
        try {
                EmployeeMapperResultMap mapper = session.getMapper(EmployeeMapperResultMap.class);
            Employee employee = mapper.getEmpAndDeptStep(1001);
            System.out.println(employee);
            System.out.println(employee.getDept());
        } finally {
            session.close();
        }
    }

    @Test
    public void testResultMapCollectionStep() throws IOException {
        SqlSessionFactory ssf = getsqlSessionFactory();
        SqlSession session = ssf.openSession();
        try {
            DepartmentMapperResultMap mapper = session.getMapper(DepartmentMapperResultMap.class);
            Department dept = mapper.getDeptAndEmpsStep(1);
            System.out.println(dept.getDepartmentName());
            System.out.println(dept.getEmps());
        }finally {
            session.close();
        }
    }
}
```



## association

###  级联

```xml
    <!-- 自定义映射
        type: 最终结果集封装的类型
        <id>: 完成主键列的映射
            column: 指定结果集的列名
            property:指定对象的属性名
        <result>:完成普通列的映射
     -->
    <select id="getEmployeeById" resultMap="MyEmp">
		select id ,last_name, email,gender from tbl_employee where id = #{id}
	</select>
    <resultMap type="com.hph.mybatis.beans.Employee" id="MyEmp">
        <id column="id" property="id"/>
        <result column="last_name" property="lastName"/>
        <result column="email" property="email"/>
        <result column="gender" property="gender"/>
    </resultMap>
```

testResultMap结果

![An2foQ.png](https://s2.ax1x.com/2019/03/19/An2foQ.png)

### Association

```xml
    <select id="getEmpAndDept" resultMap="myEmpAndDept">
          SELECT e.id eid ,  e.last_name, e.email,e.gender  , d.id did , d.dept_name
		 FROM  tbl_employee  e  , tbl_dept  d
		 WHERE e.d_id = d.id  AND e.id = #{id}
    </select>
    <resultMap type="com.hph.mybatis.beans.Employee" id="myEmpAndDept">
        <id column="eid" property="id"/>
        <result column="last_name" property="lastName"/>
        <result column="email" property="email"/>
        <result column="gender" property="gender"/>
        <!-- 级联的方式 -->
        <result column="did" property="dept.id"/>
        <result column="dept_name" property="dept.departmentName"/>
    </resultMap>
```

getEmpAndDept的结果

![AnRPOK.png](https://s2.ax1x.com/2019/03/19/AnRPOK.png)

#### 分步查询

实际的开发中，对于每个实体类都应该有具体的增删改查方法，也就是DAO层， 因此对于查询员工信息并且将对应的部门信息也查询出来的需求，就可以通过分步的方式完成查询。

①   先通过员工的id查询员工信息

②   再通过查询出来的员工信息中的外键(部门id)查询对应的部门信息. 

```xml
<!--public  Employee getEmpAndDeptStep(Integer id);-->
    <select id="getEmpAndDeptStep" resultMap="myEmpAndDeptStep">
	  	select id, last_name, email,gender ,d_id  from tbl_employee where id = #{id}
	  </select>
    <resultMap type="com.hph.mybatis.beans.Employee" id="myEmpAndDeptStep">
        <id column="id" property="id"/>
        <result column="last_name" property="lastName"/>
        <result column="email" property="email"/>
        <result column="gender" property="gender"/>
        <!-- 分步查询 -->
        <association property="dept"
                     select="com.hph.mybatis.dao.DepartmentMapperResultMap.getDeptById"
                     column="{did=d_id}" fetchType="eager">
        </association>
    </resultMap>
    <!-- association 分步查询使用延迟加载/懒加载:
            在全局配置文件中加上两个settings设置:
            <setting name="lazyLoadingEnabled" value="true"/>
          <setting name="aggressiveLazyLoading" value="false"/>
     -->
    <!-- public List<Employee>  getEmpsByDid(Integer did ); -->
    <select id="getEmpsByDid" resultType="com.hph.mybatis.beans.Employee">
        select  id,last_name,email,gender from  tbl_employee where  d_id = #{did}
    </select>
```

 #### 延迟加载

在分步查询的基础上，可以使用延迟加载来提升查询的效率，只需要在全局(mybatis-config.xml)的Settings中进行如下的配置:

```xml
<!-- 开启延迟加载 -->
<setting name="lazyLoadingEnabled" value="true"/>
<!-- 设置加载的数据是按需还是全部 -->
<setting name="aggressiveLazyLoading" value="false"/>
```

## collection

POJO中的属性可能会是一个集合对象,我们可以使用联合查询，并以级联属性的方式封装对象.使用collection标签定义对象的封装规则

```xml
    <select id="getDeptAndEmpsById" resultMap="myDeptAndEmps">
		SELECT d.id did ,d.dept_name, e.id eid, e.last_name, e.email,e.gender
		FROM tbl_dept d  LEFT OUTER JOIN  tbl_employee  e
		ON d.id = e.d_id  WHERE d.id = #{id}
	</select>
    <resultMap type="com.hph.mybatis.beans.Department" id="myDeptAndEmps">
        <id column="did" property="id"/>
        <result column="dept_name" property="departmentName"/>
        <!--
            collection: 完成集合类型的联合属性的映射
                property: 指定联合属性
                ofType: 指定集合中元素的类型
         -->
        <collection property="emps" ofType="com.hph.mybatis.beans.Employee" >
            <id column="eid" property="id"/>
            <result column="last_name" property="lastName"/>
            <result column="email" property="email"/>
            <result column="gender" property="gender"/>
        </collection>
    </resultMap>
```

![AnRPOK.png](https://s2.ax1x.com/2019/03/19/AnRPOK.png)

### 分步查询

  实际的开发中，对于每个实体类都应该有具体的增删改查方法，也就是DAO层， 因此对于查询部门信息并且将对应的所有的员工信息也查询出来的需求，就可以通过分步的方式完成查询。

①   先通过部门的id查询部门信息

②   再通过部门id作为员工的外键查询对应的部门信息. 

```xml
<select id="getDeptAndEmpsStep" resultMap="myDeptAndEmpsStep">
	 	select id , dept_name from tbl_dept where id = #{id}
	 </select>

    <resultMap  id="myDeptAndEmpsStep" type="com.hph.mybatis.beans.Department">
        <id column="id" property="id"/>
        <result column="dept_name" property="departmentName"/>
        <collection property="emps"
                    select="com.hph.mybatis.dao.EmployeeMapperResultMap.getEmpsByDid"
                    column="id">
        </collection>
    </resultMap>
```

![Anh0nP.png](https://s2.ax1x.com/2019/03/19/Anh0nP.png)

## 分步查询多列值的传递

如果分步查询时，需要传递给调用的查询中多个参数，则需要将多个参数封装成Map来进行传递，语法如下: {k1=v1, k2=v2....}

在所调用的查询方，取值时就要参考Map的取值方式，需要严格的按照封装map时所用的key来取值. 

column="{key1=column1,key2=column2}"

##  fetchType属性

 在&lt;association&gt; 和&lt;collection&gt;标签中都可以设置fetchType，指定本次查询是否要使用延迟加载。默认为 fetchType=”lazy” ,如果本次的查询不想使用延迟加载，则可设置为fetchType=”eager”.

fetchType可以灵活的设置查询是否需要使用延迟加载，而不需要因为某个查询不想使用延迟加载将全局的延迟加载设置关闭..







