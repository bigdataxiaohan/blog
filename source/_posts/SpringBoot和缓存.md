---
title: SpringBoot和缓存
date: 2019-04-09 11:08:25
tags: 
  - JSR107
categories: SpringBoot 
---

## 简介

JSR是Java Specification Requests的缩写，意思是Java 规范提案。是指向[JCP](https://baike.baidu.com/item/JCP)(Java Community Process)提出新增一个标准化技术规范的正式请求。任何人都可以提交JSR，以向Java平台增添新的API和服务。JSR已成为Java界的一个重要标准。

2012年10月26日JSR规范委员会发布了JSR 107（JCache API）的首个早期草案。自该JSR启动以来，已经过去近12年时间，因此该规范颇为Java社区所诟病，但由于目前对缓存需求越来越多，因此专家组加快了这一进度。

JCache规范定义了一种对Java对象临时在内存中进行缓存的方法，包括对象的创建、共享访问、假脱机（spooling）、失效、各JVM的一致性等，可被用于缓存JSP内最经常读取的数据，如产品目录和价格列表。利用JCACHE，多数查询的反应时间会因为有缓存的数据而加快（内部测试表明反应时间大约快15倍）。

## 内容

Java Caching定义了5个核心接口，分别是**CachingProvider**, **CacheManager**, **Cache**, **Entry** 和 **Expiry**。

•**CachingProvider**定义了创建、配置、获取、管理和控制多个**CacheManager**。一个应用可以在运行期访问多个CachingProvider。

•**CacheManager**定义了创建、配置、获取、管理和控制多个唯一命名的**Cache**，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。

•**Cache**是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。

•**Entry**是一个存储在Cache中的key-value对。

•**Expiry** 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。

![AIpCqA.png](https://s2.ax1x.com/2019/04/09/AIpCqA.png)



## 重要概念

![AIpUsJ.png](https://s2.ax1x.com/2019/04/09/AIpUsJ.png)

![AIpRLd.png](https://s2.ax1x.com/2019/04/09/AIpRLd.png)

## 环境搭建

![AIpLLj.png](https://s2.ax1x.com/2019/04/09/AIpLLj.png)

![AI9MlD.png](https://s2.ax1x.com/2019/04/09/AI9MlD.png)

### SQL

```sql

SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for department
-- ----------------------------
DROP TABLE IF EXISTS `department`;
CREATE TABLE `department` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `departmentName` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for employee
-- ----------------------------
DROP TABLE IF EXISTS `employee`;
CREATE TABLE `employee` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `lastName` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `gender` int(2) DEFAULT NULL,
  `d_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### Bean

```java
package com.hph.cache.bean;

public class Department {
	
	private Integer id;
	private String departmentName;
	
	
	public Department() {
		super();
		// TODO Auto-generated constructor stub
	}
	public Department(Integer id, String departmentName) {
		super();
		this.id = id;
		this.departmentName = departmentName;
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

```java
package com.hph.cache.bean;

public class Employee {
	
	private Integer id;
	private String lastName;
	private String email;
	private Integer gender; //性别 1男  0女
	private Integer dId;
	
	
	public Employee() {
		super();
	}

	
	public Employee(Integer id, String lastName, String email, Integer gender, Integer dId) {
		super();
		this.id = id;
		this.lastName = lastName;
		this.email = email;
		this.gender = gender;
		this.dId = dId;
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
	public Integer getdId() {
		return dId;
	}
	public void setdId(Integer dId) {
		this.dId = dId;
	}
	@Override
	public String toString() {
		return "Employee [id=" + id + ", lastName=" + lastName + ", email=" + email + ", gender=" + gender + ", dId="
				+ dId + "]";
	}
	
}
```

### 整合Mybatis

#### 配置数据源

```properties
spring.datasource.url=jdbc:mysql://192.168.1.110:3306/spring_cache
spring.datasource.data-username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

#### 数据准备

![AIiSYt.png](https://s2.ax1x.com/2019/04/09/AIiSYt.png)

## 注解版

```java
package com.hph.cache.mapper;

import com.hph.cache.bean.Employee;
import org.apache.ibatis.annotations.*;

@Mapper
public interface EmployeeMapper {

    @Select("SELECT * FROM  employee WHERE id = #{id}")
    public Employee getEmpById(Integer id);

    @Update("UPDATE employee SET lastName=#{lastName},email=#{email},gender=#{gender},d_id=#{dID} WHERE id = #{id} ")
    public  void updateEmp(Employee employee);

    @Delete("DELETE FROM employee WHERE id=#{id} ")
    public void deleteEmp(Integer id);

    @Insert("INSERT INTO employee(lastName,email,gender,d_id) VALUESE(#{lastName},#{email},#{gender},#{dId}) ")
    public void insertEmployee(Employee employee);
}
```

### 测试

```java
package com.hph.cache;

import com.hph.cache.bean.Employee;
import com.hph.cache.mapper.EmployeeMapper;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootCacheApplicationTests {
    @Autowired
    EmployeeMapper employeeMapper;

    @Test
    public void getEmpById() {
        Employee emp = employeeMapper.getEmpById(1);
        System.out.println(emp);
    }

}
```

![AIinf0.png](https://s2.ax1x.com/2019/04/09/AIinf0.png)

![AIifc8.png](https://s2.ax1x.com/2019/04/09/AIifc8.png)

![AIihjS.png](https://s2.ax1x.com/2019/04/09/AIihjS.png)

因为没有开启驼峰命名法，数据库中的字段和JavaBean中的字段未完全对应，在application.properties中配置驼峰命名。

```properties
spring.datasource.url=jdbc:mysql://192.168.1.110:3306/spring_cache
spring.datasource.username=root
spring.datasource.password=123456
#驼峰命名开启
mybatis.configuration.map-underscore-to-camel-case=true  
```

![AIiXcT.png](https://s2.ax1x.com/2019/04/09/AIiXcT.png)

## 使用缓存

添加@EnableCaching的注解

````java
package com.hph.cache;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@MapperScan("com.hph.cache.mapper")
@SpringBootApplication
@EnableCaching  //开启基于注解的缓存
public class SpringbootCacheApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootCacheApplication.class, args);
    }

}
````

开启DEBUG

```properties
spring.datasource.url=jdbc:mysql://192.168.1.110:3306/spring_cache
spring.datasource.username=root
spring.datasource.password=123456
#驼峰命名开启
mybatis.configuration.map-underscore-to-camel-case=true  
#开启debug
logging.level.com.hph.cache.mapper=debug
```

![AIFyKU.png](https://s2.ax1x.com/2019/04/09/AIFyKU.png)

![AIpQZn.png](https://s2.ax1x.com/2019/04/09/AIpQZn.png)

```java


package org.springframework.cache.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.concurrent.Callable;

import org.springframework.core.annotation.AliasFor;

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Cacheable {

	@AliasFor("cacheNames")
	String[] value() default {};

    //指定缓存的名字  
    //CacheManager管理多个组件，对缓存的真正的CRUD操作在Cache组件中，每一个缓存操作有自己唯一的名字
	@AliasFor("value")
	String[] cacheNames() default {};
	//缓存数据使用的key可以用它来指定，默认是方法的参数值。
	String key() default "";

	//主键key的生成器：可以自己指定key的生成器组件ID 
    //  key/keyGenerator  二者选一者使用
	String keyGenerator  () default "";

	//指定缓存管理器
	String cacheManager() default "";


	String cacheResolver() default "";

	//指定符合条件下才缓存
	String condition() default "";
	
	//否定缓存,当unless指定的条件为true,方法的返回值就不会被缓存,可以取到结果进行判断
	String unless() default "";


	boolean sync() default false;

}

```

| **名字**        | **位置**           | **描述**                                                     | **示例**             |
| --------------- | ------------------ | ------------------------------------------------------------ | -------------------- |
| methodName      | root object        | 当前被调用的方法名                                           | #root.methodName     |
| method          | root object        | 当前被调用的方法                                             | #root.method.name    |
| target          | root object        | 当前被调用的目标对象                                         | #root.target         |
| targetClass     | root object        | 当前被调用的目标对象类                                       | #root.targetClass    |
| args            | root object        | 当前被调用的方法的参数列表                                   | #root.args[0]        |
| caches          | root object        | 当前方法调用使用的缓存列表（如@Cacheable(value={"cache1",   "cache2"})），则有两个cache | #root.caches[0].name |
| *argument name* | evaluation context | 方法参数的名字. 可以直接 #参数名 ，也可以使用 #p0或#a0 的形式，0代表参数的索引； | #iban 、 #a0 、  #p0 |
| result          | evaluation context | 方法执行后的返回值（仅当方法执行之后的判断有效，如‘unless’，’cache put’的表达式 ’cache evict’的表达式beforeInvocation=false） | #result              |

```java
package com.hph.cache.service;

import com.hph.cache.bean.Employee;
import com.hph.cache.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    //将方法的运行结果进行缓存,如果有相同的数据直接从缓存中获取不用调用方法
    @Cacheable(cacheNames = {"emp"})
    public Employee getEmp(Integer id) {
        System.out.println("查询" + id + "号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }
}
```

```java
package com.hph.cache;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@MapperScan("com.hph.cache.mapper")
@SpringBootApplication
@EnableCaching  //开启基于注解的缓存
public class SpringbootCacheApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootCacheApplication.class, args);
    }

}
```

![Aokec6.png](https://s2.ax1x.com/2019/04/09/Aokec6.png)

![AokdHg.png](https://s2.ax1x.com/2019/04/09/AokdHg.png)

缓存生效了。

### 流程

在CacheAutoConfiguration中

```java
	static class CacheConfigurationImportSelector implements ImportSelector {

		@Override
		public String[] selectImports(AnnotationMetadata importingClassMetadata) {
			CacheType[] types = CacheType.values();
			String[] imports = new String[types.length];
			for (int i = 0; i < types.length; i++) {
				imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
			}
			return imports;
		}

	}
```

```java
public interface ImportSelector {

	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```

我们试着在CacheAutoConfiguration中打断点。看其加载的配置类。

![AoEakQ.png](https://s2.ax1x.com/2019/04/09/AoEakQ.png)

![AoZwMq.png](https://s2.ax1x.com/2019/04/09/AoZwMq.png)

![AoeKkF.png](https://s2.ax1x.com/2019/04/09/AoeKkF.png)

 ```java
//给容器中注册了一个CacheManager;ConcurrentMapCacheManager
//可以获取和创建ConcurrentMapCache类型的缓存的缓存组件
class SimpleCacheConfiguration {
	
	private final CacheProperties cacheProperties;

	private final CacheManagerCustomizers customizerInvoker;

	SimpleCacheConfiguration(CacheProperties cacheProperties,
			CacheManagerCustomizers customizerInvoker) {
		this.cacheProperties = cacheProperties;
		this.customizerInvoker = customizerInvoker;
	}

	@Bean
    //查看ConcurrentMapCacheManager
	public ConcurrentMapCacheManager cacheManager() {
		ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager();
		List<String> cacheNames = this.cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			cacheManager.setCacheNames(cacheNames);
		}
		return this.customizerInvoker.customize(cacheManager);
	}
}
 ```

```java
	public Cache getCache(String name) {
        //按照名字获取组件
		Cache cache = this.cacheMap.get(name);
		if (cache == null && this.dynamic) {
			synchronized (this.cacheMap) {
				cache = this.cacheMap.get(name);
                  //缓存如果为空 我们选择创建一个
				if (cache == null) {
					cache = createConcurrentMapCache(name);
					this.cacheMap.put(name, cache);
				}
			}
		}
		return cache;
	}

	//创建一个缓存对象
	protected Cache createConcurrentMapCache(String name) {
		SerializationDelegate actualSerialization = (isStoreByValue() ? this.serialization : null);
		return new ConcurrentMapCache(name, new ConcurrentHashMap<>(256),
				isAllowNullValues(), actualSerialization);

	}
```

```java
	private final ConcurrentMap<Object, Object> store;
```

     @Cacheable：
        1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
           （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
        2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
           key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
               SimpleKeyGenerator生成key的默认策略；
                       如果没有参数；key=new SimpleKey()；
                       如果有一个参数：key=参数的值
                       如果有多个参数：key=new SimpleKey(params)；
        3、没有查到缓存就调用目标方法；
        4、将目标方法返回的结果，放进缓存中
     
        @Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
        如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；
    
        核心：
           1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
           2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
```java
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    //将方法的运行结果进行缓存,如果有相同的数据直接从缓存中获取不用调用方法
    @Cacheable(cacheNames = {"emp"},key = "#root.methodName+'['+#id+']'")
    public Employee getEmp(Integer id) {
        System.out.println("查询" + id + "号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }
}
```

![AolZ8S.png](https://s2.ax1x.com/2019/04/09/AolZ8S.png)

### 自定义myKeyGenerator

```java
package com.hph.cache.config;

import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.lang.reflect.Method;
import java.util.Arrays;

@Configuration
public class MyCacheConfig {

    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){
        return new KeyGenerator(){

            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName()+"["+ Arrays.asList(params).toString()+"]";
            }
        };
    }
}
```

![Ao1pGT.png](https://s2.ax1x.com/2019/04/09/Ao1pGT.png)

注意要关闭`debug=true`

![Ao3E6g.png](https://s2.ax1x.com/2019/04/09/Ao3E6g.png)

```java
@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    //condition = "#a0>1 第一个参数的值>>1的时候可以进行缓存 除非a0参数是2
    @Cacheable(cacheNames = {"emp"},keyGenerator = "myKeyGenerator",condition = "#a0>0",unless = "#a0==2")
    public Employee getEmp(Integer id) {
        System.out.println("查询" + id + "号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }
}
```

![Ao8CDJ.png](https://s2.ax1x.com/2019/04/09/Ao8CDJ.png)

### CachePut

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CachePut {
//和Cacheable基本相似
	@AliasFor("cacheNames")
	String[] value() default {};

	@AliasFor("value")
	String[] cacheNames() default {};

	String key() default "";

	String keyGenerator() default "";

	String cacheManager() default "";

	String cacheResolver() default "";

	String condition() default "";

	String unless() default "";

}
```

#### 实现

```java
package com.hph.cache.service;

import com.hph.cache.bean.Employee;
import com.hph.cache.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    //condition = "#a0>1 第一个参数的值>>1的时候可以进行缓存 除非a0参数是2
    @Cacheable(cacheNames = {"emp"}/*, keyGenerator = "myKeyGenerator", condition = "#a0>0", unless = "#a0==2"*/)
    public Employee getEmp(Integer id) {
        System.out.println("查询" + id + "号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }

    /**
     * @CachePut即调用方法,又更新缓存数据 修改了数据库的某个数据, 同时更新缓存
     *运行时机:先调用目标方法,现将目标方法的结果缓存起来
     */
    @CachePut(value = "emp")
    public Employee updateEmp(Employee employee) {
        System.out.println("updateEmp"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }
}
```

```java
package com.hph.cache.service;

import com.hph.cache.bean.Employee;
import com.hph.cache.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    //condition = "#a0>1 第一个参数的值>>1的时候可以进行缓存 除非a0参数是2
    @Cacheable(cacheNames = {"emp"}/*, keyGenerator = "myKeyGenerator", condition = "#a0>0", unless = "#a0==2"*/)
    public Employee getEmp(Integer id) {
        System.out.println("查询" + id + "号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }

    /**
     * @CachePut即调用方法,又更新缓存数据 修改了数据库的某个数据, 同时更新缓存
     *运行时机:先调用目标方法,现将目标方法的结果缓存起来
     */
    @CachePut(value = "emp")
    public Employee updateEmp(Employee employee) {
        System.out.println("updateEmp"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }
}
```

```java
package com.hph.cache.controller;

import com.hph.cache.bean.Employee;
import com.hph.cache.service.EmployeeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EmployeeController {
    @Autowired
    EmployeeService employeeService;

    @GetMapping("/emp/{id}")
    public Employee getEmployee(@PathVariable("id") Integer id) {
        Employee emp = employeeService.getEmp(id);
        return emp;
    }

    @GetMapping("/emp")
    public Employee update(Employee employee) {
        Employee emp = employeeService.updateEmp(employee);
        return emp;

    }
}
```



#### 测试步骤

1. 查询1号员工查到的j结果会放在缓存中。

    key:1  value:LastName=清风丶

![AotWSx.png](https://s2.ax1x.com/2019/04/09/AotWSx.png)

2.以后查询还是之前的结果

3.更新员工信息1号员工信息【LastName=清风丶；email=`qingfeng@gmail.com`】

将方法的返回值也放进了缓存

key:传入的employee对象  value:返回的employee对象

![AoUSv6.png](https://s2.ax1x.com/2019/04/09/AoUSv6.png) 

4.查询员工

![AoaZo4.png](https://s2.ax1x.com/2019/04/09/AoaZo4.png)1号员工没有在缓存中更新。

key=“#employee。id”使用传入参数员工的id；

key=“result.id”使用返回后的id

```java
package com.hph.cache.service;

import com.hph.cache.bean.Employee;
import com.hph.cache.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    //condition = "#a0>1 第一个参数的值>>1的时候可以进行缓存 除非a0参数是2
    @Cacheable(cacheNames = {"emp"}/*, keyGenerator = "myKeyGenerator", condition = "#a0>0", unless = "#a0==2"*/)
    public Employee getEmp(Integer id) {
        System.out.println("查询" + id + "号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }

    /**
     * @CachePut即调用方法,又更新缓存数据 修改了数据库的某个数据, 同时更新缓存
     *运行时机:先调用目标方法,现将目标方法的结果缓存起来
     */
    @CachePut(value = "emp",key = "#result.id ")
    public Employee updateEmp(Employee employee) {
        System.out.println("updateEmp"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }
}
```

![Aow9K0.png](https://s2.ax1x.com/2019/04/09/Aow9K0.png)

![AowFVU.png](https://s2.ax1x.com/2019/04/09/AowFVU.png)

![AowlVO.png](https://s2.ax1x.com/2019/04/09/AowlVO.png)

查找

![AowYRA.png](https://s2.ax1x.com/2019/04/09/AowYRA.png)

已经更新过来了

### CacheEvict

EmployeeService

````java
package com.hph.cache.service;

import com.hph.cache.bean.Employee;
import com.hph.cache.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    //condition = "#a0>1 第一个参数的值>>1的时候可以进行缓存 除非a0参数是2
    @Cacheable(cacheNames = {"emp"}/*, keyGenerator = "myKeyGenerator", condition = "#a0>0", unless = "#a0==2"*/)
    public Employee getEmp(Integer id) {
        System.out.println("查询" + id + "号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }

    /**
     * @CachePut即调用方法,又更新缓存数据 修改了数据库的某个数据, 同时更新缓存
     *运行时机:先调用目标方法,现将目标方法的结果缓存起来
     */
    @CachePut(value = "emp",key = "#result.id ")
    public Employee updateEmp(Employee employee) {
        System.out.println("updateEmp"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }

    /**
     * @Cachevict :缓存清除
     * key: 指定要清除的数据
     */
    @CacheEvict(value = "emp",key = "#id")
    public void deleteEmp(Integer id){
        System.out.println("deletEmp"+id);
        employeeMapper.deleteEmpById(id);
    }

}

````

EmployeeController

```java
package com.hph.cache.controller;

import com.hph.cache.bean.Employee;
import com.hph.cache.service.EmployeeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EmployeeController {
    @Autowired
    EmployeeService employeeService;

    @GetMapping("/emp/{id}")
    public Employee getEmployee(@PathVariable("id") Integer id) {
        Employee emp = employeeService.getEmp(id);
        return emp;
    }

    @GetMapping("/emp")
    public Employee update(Employee employee) {
        Employee emp = employeeService.updateEmp(employee);
        return emp;
    }

    @GetMapping("/delemp")
    public String deleteEmp(Integer id) {
        employeeService.deleteEmp(id);
        return "删除成功";
    }
}
```

![Ao0lyq.png](https://s2.ax1x.com/2019/04/09/Ao0lyq.png)

(⊙﹏⊙)出现乱码了 先忽略吧

![Ao08mV.png](https://s2.ax1x.com/2019/04/09/Ao08mV.png)

![Ao0yTO.png](https://s2.ax1x.com/2019/04/09/Ao0yTO.png)

```java
    //删除缓存中的所有数据
    @CacheEvict(value = "emp",key = "#id",allEntries = true)
    public void deleteEmp(Integer id){
        System.out.println("deletEmp"+id);
        employeeMapper.deleteEmpById(id);
    }
```

![AoBSBV.png](https://s2.ax1x.com/2019/04/09/AoBSBV.png)

数据被全部清空

![AoBGjI.png](https://s2.ax1x.com/2019/04/09/AoBGjI.png)

```java
    //缓存默认清除才足在方法执行之后执行;如果出现异常缓存不会被清除
    @CacheEvict(value = "emp")
    public void deleteEmp(Integer id){
        System.out.println("deletEmp"+id);
      //  employeeMapper.deleteEmpById(id);
        int i = 10/0;
    }
```

![AoBRET.png](https://s2.ax1x.com/2019/04/09/AoBRET.png)

![AoBbb6.png](https://s2.ax1x.com/2019/04/09/AoBbb6.png)

2号数据库缓存也被清除掉了

![AoBvPe.png](https://s2.ax1x.com/2019/04/09/AoBvPe.png)













