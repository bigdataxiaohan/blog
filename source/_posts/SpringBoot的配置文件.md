---
title: SpringBoot的配置文件
date: 2019-03-25 16:57:21
tags: SpringBoot
categories: SpringBoot
---

## 全局配置文件

- application.properties 
- application.yml

## yml简介

yml是YAML（YAML Ain't Markup Language）语言的文件，以数据为中心，比json、xml等更适合做配置文件。

http://www.yaml.org/ 参考语法规

### 数据结构

- 对象：键值对的集合
-  数组：一组按次序排列的值 
- 字面量：单个的、不可再分的值

#### 对象

对象的一组键值对，使用冒号分隔。如：username: admin 

冒号后面跟空格来分开键值； 

{k: v}是行内写法

```yaml
person:
  last-name: lisi
  age: 18
```

#### 数组

 一组连词线（-）开头的行，构成一个数组，[]为行内写法 – 数组，对象可以组合使

```yaml
  list:
    - lisi
    - zhaoliu
```

#### 复合结构

 &nbsp; &nbsp;以上写法的任意组合都是可以

#### 字面量

 &nbsp; &nbsp;数字、字符串、布尔、日期 

 &nbsp; &nbsp;字符串 

 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;默认不使用引号

 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;可以使用单引号或者双引号，单引号会转义特殊字符 

 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;字符串可以写成多行，从第二行开始，必须有一个单空格缩进。换行符会被转为空格。

#### 文档

 &nbsp; &nbsp;多个文档用 - - - 隔

## 测试

### Person

```java
package com.hph.springboot.bean;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.List;
import java.util.Map;

@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    /**
     * <bean class = "Person">
     *     <property name="LastName" value="字面量/${key}从环境变量获取值配置文件获取值/#{SpEL}"> </property>
     *  </bean>
     */
    private  String lastName;
    private  Integer age;
    private  Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> list;
    private  Dog dog;

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getBoss() {
        return boss;
    }

    public void setBoss(Boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getList() {
        return list;
    }

    public void setList(List<Object> list) {
        this.list = list;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", list=" + list +
                ", dog=" + dog +
                '}';
    }
}

```

### Dog

```java
package com.hph.springboot.bean;

public class Dog {
    private  String name;
    private  Integer age;

    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

```

### application.yml

```yaml
person:
  last-name: 李四
  age: 18
  boss: false
  birth: 2000/12/12
  maps: {key1: value1,key2: 12}
  list:
    - lisi
    - zhaoliu
  dog:
    name: 狗狗
    age: 2
```

### Test

```java
package com.hph.springboot;

import com.hph.springboot.bean.Person;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * SpirngBoot单元测试;
 * 可以在测试期间很方便的类似编码一样进行自动注入等.
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloworldApplicationTests {

    @Autowired
    Person person;

    @Test
    public void contextLoads() {
        System.out.println(person);
    }

}
```

```tex
Person{lastName='李四', age=18, boss=false, birth=Tue Dec 12 00:00:00 CST 2000, maps={key1=value1, key2=12}, list=[lisi, zhaoliu], dog=Dog{name='狗狗', age=2}}
```

## 注解获取值对比

 @Value获取值和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

专门编写了一个javaBean来和配置文件进行映射，直接使用@ConfigurationProperties；

## 属性命名规则

- person.firstName：使用标准方式 
- person.first-name：大写用 
- person.first_name：大写用_
- PERSON_FIRST_NAME

## 配置文件注入值数据校验

```java
package com.hph.springboot.bean;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;

import javax.validation.constraints.Email;
import java.util.Date;
import java.util.List;
import java.util.Map;

@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    /**
     * <bean class="Person">
     * <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEL}"></property>
     * <bean/>
     */

    //lastName必须是邮箱格式
    @Email
    private String lastName;
    @Value("#{11-2}")
    private Integer age;
    @Value("True")
    private Boolean boss;
    private Date birth;

    private Map<String, Object> maps;
    private List<Object> list;
    private Dog dog;

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getBoss() {
        return boss;
    }

    public void setBoss(Boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getList() {
        return list;
    }

    public void setList(List<Object> list) {
        this.list = list;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", list=" + list +
                ", dog=" + dog +
                '}';
    }
}
```

![AtbEtK.png](https://s2.ax1x.com/2019/03/25/AtbEtK.png)



## 加载指定的配置文件

### person.properties

```properties
person.last-name=李四
person.age=18
person.birth=2019/3/25
person.boss=false
person.maps.k1=v1
person.maps.k2=v2
person.maps.k3=v3
person.maps.k4=14
person.lista,b,c,d
person.dog.name=dog
person.dog.age=15
```

### Person

```java
package com.hph.springboot.bean;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.List;
import java.util.Map;

@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    /**
     * <bean class="Person">
     * <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEL}"></property>
     * <bean/>
     */

    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String, Object> maps;
    private List<Object> list;
    private Dog dog;

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getBoss() {
        return boss;
    }

    public void setBoss(Boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getList() {
        return list;
    }

    public void setList(List<Object> list) {
        this.list = list;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", list=" + list +
                ", dog=" + dog +
                '}';
    }
}
```

### Test

```java
package com.hph.springboot;

import com.hph.springboot.bean.Person;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * SpirngBoot单元测试;
 * 可以在测试期间很方便的类似编码一样进行自动注入等.
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloworldApplicationTests {

    @Autowired
    Person person;

    @Test
    public void contextLoads() {
        System.out.println(person);
    }

}

```

```
Person{lastName='李四', age=18, boss=false, birth=Tue Dec 12 00:00:00 CST 2000, maps={key1=value1, key2=12, k1=v1, k4=14, k3=v3, k2=v2}, list=[lisi, zhaoliu], dog=Dog{name='狗狗', age=2}}
```

## 加载指定的Spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="helloService" class="com.hph.springboot.Service.HelloWorldService"></bean>
</beans>
```

```java
package com.hph.springboot;

import com.hph.springboot.bean.Person;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * SpirngBoot单元测试;
 * 可以在测试期间很方便的类似编码一样进行自动注入等.
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloworldApplicationTests {

    @Autowired
    ApplicationContext ioc;

    @Test
    public void testHelloService() {
        boolean b = ioc.containsBean("helloService");
        System.out.println(b);

    }

}

```

Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；

@**ImportResource**：导入Spring的配置文件，让配置文件里面的内容生效；

想让Spring的配置文件生效，加载进来；@**ImportResource**标注在一个配置类上

```java
package com.hph.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ImportResource;

//引入本地配置文件
@ImportResource(locations = {"classpath:beans.xml"})
@SpringBootApplication
public class HelloworldApplication {

   public static void main(String[] args) {
      SpringApplication.run(HelloworldApplication.class, args);
   }

}
```

结果为True

## 配置类（全注解）

SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式

1、配置类**@Configuration**------>Spring配置文件

2、使用**@Bean**给容器中添加组件

```java
package com.hph.springboot.Service;

public class HelloService {
}
```

```java
package com.hph.springboot.config;

import com.hph.springboot.Service.HelloService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Configuration :指明当前类为配置类,就是来替代之前的spring配置文件
 */
@Configuration
public class MyAppConfig {

    //将方法的发挥值添加到容器中 容器中这个组件默认的id就是方法
    @Bean
    public HelloService helloService() {
        return new HelloService();

    }
}

```

```java
package com.hph.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication
public class HelloworldApplication {

	public static void main(String[] args) {
		SpringApplication.run(HelloworldApplication.class, args);
	}

}

```

```java
package com.hph.springboot;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * SpirngBoot单元测试;
 * 可以在测试期间很方便的类似编码一样进行自动注入等.
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloworldApplicationTests {


    @Autowired
    ApplicationContext ioc;

    @Test
    public void testHelloService() {
        boolean b = ioc.containsBean("helloService");
        System.out.println("配置类@Bean给容器中添加组件了...");
        System.out.println(b);

    }
}
```

![AtLzp8.png](https://s2.ax1x.com/2019/03/25/AtLzp8.png)

## 配置文件占位符

### application.properties

```properties
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2001/12/26
person.boss=false
person.maps.k1=v1
person.maps.k2=集合
person.list=a,b,c,d,e,f
person.dog.name=${person.last-name}_狗狗
person.dog.age=3
```

### Person

```java
package com.hph.springboot.bean;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.List;
import java.util.Map;

@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    /**
     * <bean class="Person">
     * <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEL}"></property>
     * <bean/>
     */

    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String, Object> maps;
    private List<Object> list;
    private Dog dog;

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getBoss() {
        return boss;
    }

    public void setBoss(Boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getList() {
        return list;
    }

    public void setList(List<Object> list) {
        this.list = list;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", list=" + list +
                ", dog=" + dog +
                '}';
    }
}
```

### Dog

```java
package com.hph.springboot.bean;

public class Dog {
    private  String name;
    private  Integer age;

    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

### Test

```java
package com.hph.springboot;

import com.hph.springboot.bean.Person;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * SpirngBoot单元测试;
 * 可以在测试期间很方便的类似编码一样进行自动注入等.
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloworldApplicationTests {

    @Autowired
    Person person;

    @Test
    public void contextLoads() {
        System.out.println(person);
    }
}

```

### 结果

```
Person{lastName='张三0563bb3c-32a3-4905-96f6-5b98f95e3f91', age=952443557, boss=false, birth=Wed Dec 26 00:00:00 CST 2001, maps={k1=v1, k2=集合}, list=[a, b, c, d, e, f], dog=Dog{name='张三68aa9e66-24d1-4134-be1a-935c598fe59c_狗狗', age=3}}
```

## 多环境支持

我们在主配置文件编写的时候，文件名可以是   application-{profile}.properties或者是yml配置文件 默认使用application.properties的配置；

```yaml
#默认8081
server:
  port: 8081
spring:
  profiles:
    active: prod #启动dev环境

---
server:
  port: 8080
spring:
  profiles: dev
---
server:
  port: 80
spring:
  profiles: prod #指定属于哪个环境
```

### 激活指定profile

- 在配置文件中指定  spring.profiles.active=dev
-  命令行：java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；可以直接在测试的时候，配置传入命令行参数
- 虚拟机参数；-Dspring.profiles.active=dev

## 配置文件加载

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

–file:./config/

–file:./

–classpath:/config/

–classpath:/

优先级由高到底，高优先级的配置会覆盖低优先级的配置；

SpringBoot会从这四个位置全部加载主配置文件；**互补配置**；

我们还可以通过`spring.config.location`来改变默认的配置文件位置

**项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；**

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.properties

## 外部配置加载顺序

**SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置**

**1.命令行参数**

所有的配置都可以在命令行上进行指定

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc

多个配置用空格分开； --配置项=值

2.来自java:comp/env的JNDI属性

3.Java系统属性（System.getProperties()）

4.操作系统环境变量

5.RandomValuePropertySource配置的random.*属性值

**由jar包外向jar包内进行寻找；**

**优先加载带profile**

**6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

**7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

**再来加载不带profile**

**8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件**

**9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件**

10.@Configuration注解类上的@PropertySource

11.通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源；

[参考官方文档](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config)












