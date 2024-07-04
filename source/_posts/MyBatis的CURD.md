---
title: MyBatis的CURD
date: 2019-03-18 19:37:19
tags: Mybatis
categories: JavaWeb
---

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

### EmployeeMapper

```java

import com.hph.mybatis.beans.Employee;
import org.apache.ibatis.annotations.MapKey;
import org.apache.ibatis.annotations.Param;

import java.util.List;
import java.util.Map;

public interface EmployeeMapper {
    //根据id查询员工
    public Employee getEmployeeById(Integer id);

    //添加一个新的员工
    public void addEmployee(Employee employee);

    //修改一个员工
    public void updateEmployee(Employee employee);

    //删除一个员工
    public Integer deleteEmployeeById(Integer id);

    //查询对象通过两个参数
    public Employee getEmployeeByIdAndLastName(@Param("id") Integer id, @Param("lastName") String lastNmae);

    //查询一个集合
    public Employee getEmployeeByMap(Map<String, Object> map);

    //查询多个数据返回一个对象的集合
    public List<Employee>  getEmps();

    //查询单条数据返回一个Map
    public Map<String,Object> getEmployeeByIdReturnMap(Integer id);

    //查询多条数据返回一个Map
    @MapKey("id")  //指定使用对象那个属性作为Map的key
    public Map<Integer,Employee> getEmpsRetrunMap();

}
```


```xml
<?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE mapper  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--配置SQL映射-->
<mapper namespace="com.hph.mybatis.dao.EmployeeMapper">
    
    <select id="selectEmployee" resultType="com.hph.mybatis.beans.Employee">
		select id, last_name , email, gender from tbl_employee where id = #{id}
    </select>
    <!--public Employee getEmployeeById(Integer id);-->
    <select id="getEmployeeById" resultType="com.hph.mybatis.beans.Employee">
		select id, last_name , email, gender from tbl_employee where id = #{id}
    </select>
    <!--public void addEmployee(Employee employee);
  parameterType:指定ca参数类型,可以省略不写
  keyProperty:指定用对象的那个属性ba保存mybatisf返回的主键值
    -->
    <!--告诉mybatis:需要使用主键自增的方式-->
    <insert id="addEmployee" useGeneratedKeys="true" keyProperty="id">
  insert  into  tbl_employee(last_name,email,gender) values (#{lastName},#{email},#{gender})
    </insert>

    <update id="updateEmployee">
	update tbl_employee set
		last_name = #{lastName},
		email = #{email},
		gender = #{gender}
		where id = #{id}
    </update>

    <delete id="deleteEmployeeById">
        delete  from  tbl_employee where id = #{id}
    </delete>
    <select id="getEmployeeByIdAndLastName" resultType="com.hph.mybatis.beans.Employee">
		select id, last_name lastName, email, gender from tbl_employee where id = #{id} and last_name = #{lastName}
    </select>
    <select id="getEmployeeByMap" resultType="com.hph.mybatis.beans.Employee">

		select id, last_name , email, gender from ${tableName} where id = ${id} and last_name = #{ln}
    </select>
    <!--public List<Employee>  getEmps()
    resultType结果集的封装类型
    ;-->
    <select id="getEmps" resultType="com.hph.mybatis.beans.Employee">
		select id, last_name , email, gender from tbl_employee
    </select>
    <select id="getEmployeeByIdReturnMap" resultType="java.util.Map">

		select id, last_name , email, gender from tbl_employee where  id = #{id}

    </select>
    <select id="getEmpsRetrunMap" resultType="java.util.Map">
		select id, last_name , email, gender from tbl_employee
    </select>

</mapper>
```

###  EmployeeMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE mapper  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--配置SQL映射-->
<mapper namespace="com.hph.mybatis.dao.EmployeeMapper">

    <!--public Employee getEmployeeById(Integer id);-->
    <select id="getEmployeeById" resultType="com.hph.mybatis.beans.Employee">
		select id, last_name , email, gender from tbl_employee where id = #{id}
    </select>
    <!--public void addEmployee(Employee employee);
  parameterType:指定ca参数类型,可以省略不写
  keyProperty:指定用对象的那个属性ba保存mybatisf返回的主键值
    -->
    <!--告诉mybatis:需要使用主键自增的方式-->
    <insert id="addEmployee" useGeneratedKeys="true" keyProperty="id">
  insert  into  tbl_employee(last_name,email,gender) values (#{lastName},#{email},#{gender})
    </insert>


    <!--public void updateEmployee(Employee employee);-->
    <update id="updateEmployee">
	update tbl_employee set
		last_name = #{lastName},
		email = #{email},
		gender = #{gender}
		where id = #{id}
    </update>

    <!--public Integer deleteEmployeeById(Integer id);-->
    <delete id="deleteEmployeeById">
        delete  from  tbl_employee where id = #{id}
    </delete>
    <!--    public Employee getEmployeeByIdAndLastName(@Param("id") Integer id, @Param("lastName") String lastNmae);
-->
    <select id="getEmployeeByIdAndLastName" resultType="com.hph.mybatis.beans.Employee">
		select id, last_name lastName, email, gender from tbl_employee where id = #{id} and last_name = #{lastName}
    </select>
    <!--public Employee getEmployeeByMap(Map<String, Object> map);-->
    <select id="getEmployeeByMap" resultType="com.hph.mybatis.beans.Employee">

		select id, last_name , email, gender from ${tableName} where id = ${id} and last_name = #{ln}
    </select>
    <!--public List<Employee>  getEmps()
    resultType结果集的封装类型
    ;-->
    <!--public List<Employee>  getEmps();-->
    <select id="getEmps" resultType="com.hph.mybatis.beans.Employee">
		select id, last_name , email, gender from tbl_employee
    </select>
    <!--public Map<String,Object> getEmployeeByIdReturnMap(Integer id);-->
    <select id="getEmployeeByIdReturnMap" resultType="java.util.Map">

		select id, last_name , email, gender from tbl_employee where  id = #{id}

    </select>
    <!--public Map<Integer,Employee> getEmpsRetrunMap();-->
    <select id="getEmpsRetrunMap" resultType="java.util.Map">
		select id, last_name , email, gender from tbl_employee
    </select>
</mapper>
```

### TestMybatisMapper

```java
package com.hph.mybatis.test;

import com.hph.mybatis.beans.Employee;
import com.hph.mybatis.dao.EmployeeMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.Map;

public class TestMybatisMapper {
    @Test
    public void testSqlsessionFactory() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        System.out.println(sqlSessionFactory);
        SqlSession session = sqlSessionFactory.openSession();
        System.out.println(session);
        try {
            Employee employee = session.selectOne("selectEmployee", 5);
            System.out.println(employee);
        } finally {
            session.close();
        }
    }

    @Test
    public void testHelloWorldMapper() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession session = sqlSessionFactory.openSession();
        try {
            //Mapper接口 dao接口
            EmployeeMapper dao = session.getMapper(EmployeeMapper.class);
            Employee employee = dao.getEmployeeById(1001);
            System.out.println(employee);
        } finally {
            session.close();
        }
    }

    public SqlSessionFactory getsqlSessionFactory() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);

        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        return sqlSessionFactory;
    }

    @Test
    public void testCRUD() throws IOException, SQLException {
        SqlSessionFactory ssf = getsqlSessionFactory();
        SqlSession session = ssf.openSession(true);
        try {
            //获取代理实现类对象 mapper接口的代理实现类对象
            EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
            //添加
            Employee employee = new Employee(null, "清风笑_Test", "qfx@qq.com", 1);
            mapper.addEmployee(employee);
            System.out.println("返回的键值" + employee.getId());

            /**
             *      Connection conn = null;
             *             PreparedStatement ps = conn.prepareStatement("sql", PreparedStatement.RETURN_GENERATED_KEYS);
             *
             *             ps.executeUpdate();
             *             ps.getGeneratedKeys();
             *             jdbc获取新插入的组件的数值
             */
           //修改
            Employee employee1 = new Employee(null,"清风_笑丶", "qfx@gmail.com", 1);

            mapper.updateEmployee(employee1);
            //删除
         //   Integer count = mapper.deleteEmployeeById(1001);
           // System.out.println(count);
        } finally {
            session.close();
        }
    }

    @Test
    public void testParameter() throws IOException {
        SqlSessionFactory ssf = getsqlSessionFactory();
        SqlSession session = ssf.openSession(true);
        try {
            EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
            //   Employee employee = mapper.getEmployeeByIdAndLastName(5, "清风_笑丶");
            Map<String, Object> map = new HashMap<String, Object>();
            map.put("id", "5");
            map.put("ln", "清风_笑丶");
            map.put("tableName", "tbl_employee");

            Employee employee = mapper.getEmployeeByMap(map);
            System.out.println(employee);
        } finally {
            session.close();
        }
    }

    @Test
    public void tetSelect() throws IOException {
        SqlSessionFactory ssf = getsqlSessionFactory();
        SqlSession session = ssf.openSession(true);
        try {
            EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
            //List<Employee> emps = mapper.getEmps();
            //System.out.println(emps);
            //Map<String, Object> map = mapper.getEmployeeByIdReturnMap(5);
            //System.out.println(map);
            Map<Integer, Employee> map = mapper.getEmpsRetrunMap();
            System.out.println(map);
        } finally {
            session.close();
        }
    }
}
```

## 结果

testHelloWorldMapper 

![AmzojK.png](https://s2.ax1x.com/2019/03/18/AmzojK.png)

addEmployee

添加数据

![AnpOSI.png](https://s2.ax1x.com/2019/03/18/AnpOSI.png)

![AnpbYd.png](https://s2.ax1x.com/2019/03/18/AnpbYd.png)

修改数据

![AnPBOP.png](https://s2.ax1x.com/2019/03/18/AnPBOP.png)

![AnP2Wj.png](https://s2.ax1x.com/2019/03/18/AnP2Wj.png)

删除数据

![AnPfln.png](https://s2.ax1x.com/2019/03/18/AnPfln.png)

![AnPhyq.png](https://s2.ax1x.com/2019/03/18/AnPhyq.png)

testParameter

![AnFHz9.png](https://s2.ax1x.com/2019/03/18/AnFHz9.png)

tetSelect

无参数

![AnkFsI.png](https://s2.ax1x.com/2019/03/18/AnkFsI.png)

有参数

![AnkKzj.png](https://s2.ax1x.com/2019/03/18/AnkKzj.png)

##  主键生成方式、获取主键值

### 主键生成方式

支持主键自增，例如MySQL数据库

不支持主键自增，例如Oracle数据库

### 获取主键值

数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），则可以设置 useGeneratedKeys=”true”，然后再把 keyProperty设置到目标属性上。

```xml
<insert id="insertEmployee" 	
parameterType="com.hph.mybatis.beans.Employee"  
			databaseId="mysql"
			useGeneratedKeys="true"
			keyProperty="id">
		insert into tbl_employee(last_name,email,gender) values(#{lastName},#{email},#{gender})
</insert>
```

### 参数传递

#### 参数传递的方式

- 单个参数: 可以接受基本类型，对象类型。这种情况MyBatis可直接使用这个参数，不需要经过任 何处理。
- 多个参数: 任意多个参数，都会被MyBatis重新包装成一个Map传入。Map的key是param1，param2，或者0，1…，值就是参数的值
- 命名参数: 为参数使用@Param起一个名字，MyBatis就会将这些参数封装进map中，key就是我们自己指定的名
- POJO: 当这些参数属于我们业务POJO时，直接传递POJO
- Map : 我们也可以封装多个参数为map，直接传递
- Collection/Array : 会被MyBatis封装成一个map传入, Collection对应的key是collection,Array对应的key是array. 如果确定是List集合，key还可以是list.

#### 参数传递源码

```java
public Object getNamedParams(Object[] args) {
    final int paramCount = names.size();
    if (args == null || paramCount == 0) {
      return null;
    } else if (!hasParamAnnotation && paramCount == 1) {
      return args[names.firstKey()];
    } else {
      final Map<String, Object> param = new ParamMap<Object>();
      int i = 0;
      for (Map.Entry<Integer, String> entry : names.entrySet()) {
        param.put(entry.getValue(), args[entry.getKey()]);
        // add generic param names (param1, param2, ...)
        final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
        // ensure not to overwrite parameter named with @Param
        if (!names.containsValue(genericParamName)) {
          param.put(genericParamName, args[entry.getKey()]);
        }
        i++;
      }
      return param;
    }
  }
```

### 参数处理

参数位置支持的属性: javaType、jdbcType、mode、numericScale、resultMap、typeHandler、jdbcTypeName、expression

实际上通常被设置的是：可能为空的列名指定 jdbcType ,例如:

```xml
insert into orcl_employee(id,last_name,email,gender) values(employee_seq.nextval,#{lastName, ,jdbcType=NULL },#{email},#{gender})可以在全局配置文件指定为null 也可以像上面这面这样单独配置
```

### 参数获取方式

```xml
#{key}：可取普通类型、POJO类型、多个参数、集合类型
获取参数的值，预编译到SQL中。安全。Preparedstatement
${key}：可取单个普通类型、POJO类型、多个参数、集合类型
注意：取单个普通类型的参数，${}中不能随便写 必须用_parameter  _parameter是Mybatis的内置参数获取参数的值，拼接到SQL中。有SQL注入问题。Statement ORDER BY ${name}

原则：能用#{}取值就优先使用#{} 解决不了的使用${}
		e.g.原生的JDBC不支持占位符的地方 就可以使用${}
		e.g.Select column1,column2…from 表 where 条件 group by 组表示 having 条件 
				order by 排序字段 desc/asc  limit X，X；
```

###  select查询的几种情况

查询单行数据返回单个对象

```java
public Employee getEmployeeById(Integer id );
```


查询多行数据返回对象的集合

```java
public List<Employee> getAllEmps();
```

查询单行数据返回Map集合

```java
public Map<String,Object> getEmployeeByIdReturnMap(Integer id );
```


查询多行数据返回Map集合

```java
@MapKey("id") // 指定使用对象的哪个属性来充当map的key(对象的属性，而不是数据库的列)
public Map<Integer,Employee>  getAllEmpsReturnMap();
```

## resultType自动映射

- autoMappingBehavior默认是PARTIAL，开启自动映射的功能。唯一的要求是列名和javaBean属性名一致
- 如果autoMappingBehavior设置为null则会取消自动映射
- 数据库字段命名规范，POJO属性符合驼峰命名法，如A_COLUMNaColumn，我们可以开启自动驼峰命名规则映射功能，mapUnderscoreToCamelCase=true

缺点：多表查询 完成不了































