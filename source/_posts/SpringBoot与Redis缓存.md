---
title: SpringBoot与Redis缓存
date: 2019-04-10 15:41:37
tags:  Redis
categories: SpringBoot 
---

## 准备

在Docker安装Redis

![ATwKnP.png](https://s2.ax1x.com/2019/04/10/ATwKnP.png)

![ATw8hQ.png](https://s2.ax1x.com/2019/04/10/ATw8hQ.png)

连接成功

![ATw639.png](https://s2.ax1x.com/2019/04/10/ATw639.png)

对于Redis不熟悉的同学可以在本站搜索Redis的文章阅读。

## 整合Redis

在pom文件中加入

```xml
 <!--引入Redis-->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```java
@Configuration
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
    //简化操作KV 都是对象的
	public RedisTemplate<Object, Object> redisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
    //简化操作字符串的
	public StringRedisTemplate stringRedisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

}
```

## 准备

```java
package com.hph.cache;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootCacheApplicationTests {

    @Autowired
    StringRedisTemplate stringRedisTemplate;

    @Autowired
    RedisTemplate redisTemplate;

    /**
     * Redis常用的数据类型
     * String(字符串)、List(列表)、Set(集合)、Hash(散列)、(有序集合)
     * String（字符串）、List（列表）、Set（集合）、Hash（散列）、ZSet（有序集合）
     * stringRedisTemplate.opsForValue()[String（字符串）]
     * stringRedisTemplate.opsForList()[List（列表）]
     * stringRedisTemplate.opsForSet()[Set（集合）]
     * stringRedisTemplate.opsForHash()[Hash（散列）]
     * stringRedisTemplate.opsForZSet()[ZSet（有序集合）]
     */
    @Test
    public void redis() {
        stringRedisTemplate.opsForValue().append("msg","hello");

    }
}
```

![ATsOk6.png](https://s2.ax1x.com/2019/04/10/ATsOk6.png)

### 读取数据

更新原有数据

![ATyCnA.png](https://s2.ax1x.com/2019/04/10/ATyCnA.png)

```java
    @Test
    public void getmesg() {
        String msg = stringRedisTemplate.opsForValue().get("msg");
        System.out.println(msg);
    }
```

![ATyMBn.png](https://s2.ax1x.com/2019/04/10/ATyMBn.png)

```java
@Test
public void listops(){
     stringRedisTemplate.opsForList().leftPush("mylist","1");
     stringRedisTemplate.opsForList().leftPush("mylist","2");
     stringRedisTemplate.opsForList().leftPush("mylist","3");
     stringRedisTemplate.opsForList().leftPush("mylist","4");
}
```

![ATyRud.png](https://s2.ax1x.com/2019/04/10/ATyRud.png)

```java
    @Test
    public void cachObject(){
        Employee empById = employeeMapper.getEmpById(1);
        redisTemplate.opsForValue().set("emp-001",empById);
    }
```

![AT6K2D.png](https://s2.ax1x.com/2019/04/10/AT6K2D.png)

### 序列化

报错Employee需要序列化。

```java
public class Employee implements Serializable 
```

![AT6Dqs.md.png](https://s2.ax1x.com/2019/04/10/AT6Dqs.md.png)

RedisTemplate默认的是JdkSerializationRedisSerializer的序列化器。

```java
	if (defaultSerializer == null) {
			defaultSerializer = new JdkSerializationRedisSerializer(
					classLoader != null ? classLoader : this.getClass().getClassLoader());
		}
```

我们可以使用RedisSerializer的序列化器设置默认的序列化器。

```java
	public void setDefaultSerializer(RedisSerializer<?> serializer) {
		this.defaultSerializer = serializer;
	}
```



![ATcs6e.png](https://s2.ax1x.com/2019/04/10/ATcs6e.png)

#### 自定义序列化器

```java
package com.hph.cache.config;

import com.hph.cache.bean.Employee;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

import java.net.UnknownHostException;

@Configuration
public class MyRedisConfig {

    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> ser = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(ser);
        return template;
    }
}
```

```java
    @Autowired
    RedisTemplate<Object, Employee> empRedisTemplate;
    @Test
    public void ObjectToJsonRedis(){
        Employee empById = employeeMapper.getEmpById(1);
        //默认保存对象使用jdk序列化机制，序列化的数据保存在redis中
        empRedisTemplate.opsForValue().set("emp-001",empById);
    }
```

成功

![ATg63T.png](https://s2.ax1x.com/2019/04/10/ATg63T.png)

好的我们开始运行以下看看使用Redis缓存的效果。

```java
package com.hph.cache.service;

import com.hph.cache.bean.Employee;
import com.hph.cache.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.Caching;
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    //condition = "#a0>1 第一个参数的值>>1的时候可以进行缓存 除非a0参数是2
    @Cacheable(cacheNames = {"emp"},keyGenerator = "myKeyGenerator")
    public Employee getEmp(Integer id) {
        System.out.println("查询" + id + "号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }

    /**
     * @CachePut即调用方法,又更新缓存数据 修改了数据库的某个数据, 同时更新缓存
     * 运行时机:先调用目标方法,现将目标方法的结果缓存起来
     */
    @CachePut(value = "emp", key = "#result.id ")
    public Employee updateEmp(Employee employee) {
        System.out.println("updateEmp" + employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }

    /**
     * @Cachevict :缓存清除
     * key: 指定要清除的数据
     */
    //缓存默认清除才足在方法执行之后执行;如果出现异常缓存不会被清除
    @CacheEvict(value = "emp", beforeInvocation = true)
    public void deleteEmp(Integer id) {
        System.out.println("deletEmp" + id);
        //  employeeMapper.deleteEmpById(id);
        int i = 10 / 0;
    }
}
```

###  流程分析

原理：CacheManager==某一个Cache缓存组件来给实际给缓存中存取得数据，比如我们使用Redis缓存，那么SimpleCacheManager就不匹配进而不能使用，从而使用Redis

![ATbUJg.png](https://s2.ax1x.com/2019/04/10/ATbUJg.png)

![ATb6oT.png](https://s2.ax1x.com/2019/04/10/ATb6oT.png)

引入Redis的starter，容器中保存的是RedisCcheManager；

RedisCacheManager帮我们创建RedisCache作为缓存组件，RedisCache通过Redis缓存数据。

 ![ATq81J.png](https://s2.ax1x.com/2019/04/10/ATq81J.png)

我们可以看到SQL只执行了一次。

![ATqtn1.png](https://s2.ax1x.com/2019/04/10/ATqtn1.png)

Redis中已经缓存我们查询过的数据。

默认保留数据的K-V都是Object；利用序列化保存可以保存为Json。

引入了redis的stater，cacheManager变为RedisCacheManager；默认创建的RedisCacheManager

```java
	@Bean
	public RedisCacheManager cacheManager(RedisTemplate<Object, Object> redisTemplate) {
		RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
		cacheManager.setUsePrefix(true);
		List<String> cacheNames = this.cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			cacheManager.setCacheNames(cacheNames);
		}
		return this.customizerInvoker.customize(cacheManager);
	}
```

RedisCacheManager操作Redis使用的是RedisTemplate默认使用的是JdkSerializationRedisSerializer

```java
	if (defaultSerializer == null) {

			defaultSerializer = new JdkSerializationRedisSerializer(
					classLoader != null ? classLoader : this.getClass().getClassLoader());
		}
```

### 缓存管理器

接下来我们定制以下缓存管理器。

```java
package com.hph.cache.config;

import com.hph.cache.bean.Employee;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

import java.net.UnknownHostException;

@Configuration
public class MyRedisConfig {
    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> ser = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(ser);
        return template;
    }

    //CacheManagerCustomizers可以定制缓存的一些规则。
    @Bean
    public RedisCacheManager  employeeCacheManager(RedisTemplate<Object, Employee> empRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }

}
```

而创建缓存管理器的条件是。

```java
@AutoConfigureAfter(RedisAutoConfiguration.class)
@ConditionalOnBean(RedisTemplate.class)
@ConditionalOnMissingBean(CacheManager.class)  //没有CacheManager的时候创建RedisCacheConfiguration
@Conditional(CacheCondition.class)
class RedisCacheConfiguration {
```

运行结果。

![ATX81g.png](https://s2.ax1x.com/2019/04/10/ATX81g.png)

我们在创建一个关于Department的缓存查询。

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
package com.hph.cache.mapper;

import com.hph.cache.bean.Department;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

@Mapper
public interface DeptMapper {
    @Select("SELECT * FROM department WHERE id = #{id}")
    Department getDeptById(Integer id);
}
```

```java
package com.hph.cache.service;

import com.hph.cache.bean.Department;
import com.hph.cache.mapper.DeptMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class DeptService {
    @Autowired
    DeptMapper deptMapper;

    @Cacheable(cacheNames = "dept")
    public Department getDeptById(Integer id) {
        System.out.println("查询部门" + id);
        Department department = deptMapper.getDeptById(id);

        return department;
    }
}

```

```java
package com.hph.cache.controller;

import com.hph.cache.bean.Department;
import com.hph.cache.service.DeptService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DeptContorller {

    @Autowired
    DeptService deptService;

    @GetMapping("/dept/{id}")
    public Department getDept(@PathVariable("id") Integer id) {
        return deptService.getDeptById(id);
    }
}
```

![ATjaKH.png](https://s2.ax1x.com/2019/04/10/ATjaKH.png)

![ATjBVI.png](https://s2.ax1x.com/2019/04/10/ATjBVI.png)

然而当我们再次请求缓存的时候。

![ATjDat.png](https://s2.ax1x.com/2019/04/10/ATjDat.png)

5个属性要映射，这是因为我们自定义的那个缓存管理器是属于员工的而不是部门，因此字段对应不上去。同时我们也发现了缓存数据第一次可以存入Redis，第二次从缓存中查询就不能够反序列化回来了。这是因为当我们存储dept的json数据时，CacheManager默认使用的是RedisTemplate&lt;Object,Employee&gt;操作Redis，只能将Employee数据反序列化。

我们需要配置以下缓存管理器。

```java
   @Bean
    public RedisTemplate<Object, Department> deptRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Department> template = new RedisTemplate<Object, Department>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Department> ser = new Jackson2JsonRedisSerializer<Department>(Department.class);
        template.setDefaultSerializer(ser);
        return template;
    }

    @Bean
    public RedisCacheManager  deptCacheManager(RedisTemplate<Object, Department> deptRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(deptRedisTemplate);
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }
```

给Employee Dept分别指定缓存类别

```java
@CacheConfig(cacheNames = "emp",cacheManager = "employeeCacheManager")
@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;
    ....
}
```

```java
@Service
public class DeptService {
    @Autowired
    DeptMapper deptMapper;

    @Cacheable(cacheNames = "dept",cacheManager = "deptCacheManager")
    public Department getDeptById(Integer id) {
        System.out.println("查询部门" + id);
        Department department = deptMapper.getDeptById(id);
        return department;
    }
}
```

![ATzTit.png](https://s2.ax1x.com/2019/04/10/ATzTit.png)

在MyRedisConfig中指定一个主类

```java
    @Primary
    @Bean
    public RedisCacheManager  employeeCacheManager(RedisTemplate<Object, Employee> empRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }
```

我们把Rdis的数据清空下

![A79E5R.png](https://s2.ax1x.com/2019/04/10/A79E5R.png)

由于默认指定的是employeeCacheManager我们可以在

```java
@CacheConfig(cacheNames = "emp")EmployeeService上省略。
@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;
```

在实际开发中不应该将employeeCacheManager作为默认指定，应该将RedisCacheManager作为默认指定。@Primary将某个缓存管理器作为默认的 

另外我们可以选择编码的方式l来配置缓存管理器。

在DeptService中。

```java
    @Qualifier("deptCacheManager")
    @Autowired
    RedisCacheManager deptCacheManager;

    public Department getDeptById(Integer id) {
        System.out.println("查询部门" + id);
        Department department = deptMapper.getDeptById(id);
        //获取某个缓存
        Cache dept = deptCacheManager.getCache("dept");
        dept.put("缓存操作1", department);
        return department;
    }
```

![A7Ptu8.png](https://s2.ax1x.com/2019/04/10/A7Ptu8.png)