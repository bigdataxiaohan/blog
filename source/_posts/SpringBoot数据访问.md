---
title: SpringBoot数据访问
date: 2019-04-08 15:52:01
tags: SpringBoot
categories: SpringBoot
---

## 准备

![A4Lxr6.png](https://s2.ax1x.com/2019/04/08/A4Lxr6.png)

![A4OpVO.png](https://s2.ax1x.com/2019/04/08/A4OpVO.png)

在Maven中会多依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
```

application.yml的配置文件关于JDBC的
```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.1.110:3306/jdbc
    driver-class-name: com.mysql.jdbc.Driver
```

## 测试

```java
package com.hph.springboot;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootDataJdbcApplicationTests {

    @Autowired
    DataSource dataSource;
    @Test
    public void contextLoads() throws SQLException {
        System.out.println(dataSource.getClass());
        Connection connection = dataSource.getConnection();
        System.out.println(connection);

    }

}
```

![A4O0JJ.png](https://s2.ax1x.com/2019/04/08/A4O0JJ.png)

默认是用org.apache.tomcat.jdbc.pool.DataSource作为数据源；数据源的相关配置都在DataSourceProperties里面；

## 自动配置

```java
/**
	 * Tomcat Pool DataSource configuration.
	 */
	@Configuration
	@ConditionalOnClass(org.apache.tomcat.jdbc.pool.DataSource.class)
	@ConditionalOnMissingBean(DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type",
			havingValue = "org.apache.tomcat.jdbc.pool.DataSource", matchIfMissing = true)
	//默认
	static class Tomcat {
		@Bean
		@ConfigurationProperties(prefix = "spring.datasource.tomcat")
		public org.apache.tomcat.jdbc.pool.DataSource dataSource(
				DataSourceProperties properties) {
			org.apache.tomcat.jdbc.pool.DataSource dataSource = createDataSource(
					properties, org.apache.tomcat.jdbc.pool.DataSource.class);
			DatabaseDriver databaseDriver = DatabaseDriver
					.fromJdbcUrl(properties.determineUrl());
			String validationQuery = databaseDriver.getValidationQuery();
			if (validationQuery != null) {
				dataSource.setTestOnBorrow(true);
				dataSource.setValidationQuery(validationQuery);
			}
			return dataSource;
		}
	}
```

1、参考DataSourceConfiguration，根据配置创建数据源，默认使用Tomcat连接池；可以使用spring.datasource.type指定自定义的数据源类型；

2、SpringBoot默认可以支持；

```text
org.apache.tomcat.jdbc.pool.DataSource    HikariDataSource   BasicDataSource、
```

3、自定义数据源类型

```java
/**
 * Generic DataSource configuration.
 */
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type")
static class Generic {

   @Bean
   public DataSource dataSource(DataSourceProperties properties) {
       //使用DataSourceBuilder创建数据源，利用反射创建响应type的数据源，并且绑定相关属性
      return properties.initializeDataSourceBuilder().build();
   }
}
```

4、`DataSourceInitializer：ApplicationListener`

runSchemaScripts();运行建表语句；

```java
	private void runSchemaScripts() {
		List<Resource> scripts = getScripts("spring.datasource.schema",
				this.properties.getSchema(), "schema");
		if (!scripts.isEmpty()) {
			String username = this.properties.getSchemaUsername();
			String password = this.properties.getSchemaPassword();
			runScripts(scripts, username, password);
			try {
				this.applicationContext
						.publishEvent(new DataSourceInitializedEvent(this.dataSource));
				// The listener might not be registered yet, so don't rely on it.
				if (!this.initialized) {
					runDataScripts();
					this.initialized = true;
				}
			}
			catch (IllegalStateException ex) {
				logger.warn("Could not send event to complete DataSource initialization ("
						+ ex.getMessage() + ")");
			}
		}
	}
```

runDataScripts();运行插入数据的sql语句；

```java
	private void runDataScripts() {
		List<Resource> scripts = getScripts("spring.datasource.data",
				this.properties.getData(), "data");
		String username = this.properties.getDataUsername();
		String password = this.properties.getDataPassword();
		runScripts(scripts, username, password);
	}
```

```java
//获取列表
private List<Resource> getScripts(String propertyName, List<String> resources,
                                  //fallback 就是schema
			String fallback) {
		if (resources != null) {
			return getResources(propertyName, resources, true);
		}
		String platform = this.properties.getPlatform();
		List<String> fallbackResources = new ArrayList<String>();
    	//如果获不到则从类路径下寻找".sql"文件
		fallbackResources.add("classpath*:" + fallback + "-" + platform + ".sql");
		fallbackResources.add("classpath*:" + fallback + ".sql");
		return getResources(propertyName, fallbackResources, false);
	}
```



```reStructuredText
schema-*.sql、data-*.sql
默认规则：schema.sql，schema-all.sql；
可以使用   
	  schema:
      - classpath:department.sql
      指定位置
```

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
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

未启动Run方法之前

![A4vsQs.png](https://s2.ax1x.com/2019/04/08/A4vsQs.png)

![A4vTyR.png](https://s2.ax1x.com/2019/04/08/A4vTyR.png)

![A4vqw6.png](https://s2.ax1x.com/2019/04/08/A4vqw6.png)

如果你想运行指定的sql文件可以在配置文件中指定

![A4xNc9.png](https://s2.ax1x.com/2019/04/08/A4xNc9.png)

执行成功

## JdbcTemplate

操作数据库：自动配置了JdbcTemplate操作数据库

```java
@Configuration
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
@ConditionalOnSingleCandidate(DataSource.class)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
public class JdbcTemplateAutoConfiguration {

	private final DataSource dataSource;

	public JdbcTemplateAutoConfiguration(DataSource dataSource) {
		this.dataSource = dataSource;
	}

	@Bean
	@Primary
	@ConditionalOnMissingBean(JdbcOperations.class)
	public JdbcTemplate jdbcTemplate() {
		return new JdbcTemplate(this.dataSource);
	}

	@Bean
	@Primary
	@ConditionalOnMissingBean(NamedParameterJdbcOperations.class)
	public NamedParameterJdbcTemplate namedParameterJdbcTemplate() {
		return new NamedParameterJdbcTemplate(this.dataSource);
	}
}
```

## 数据库准备

![A5pt1g.png](https://s2.ax1x.com/2019/04/08/A5pt1g.png)

 ![A5pc3F.png](https://s2.ax1x.com/2019/04/08/A5pc3F.png)

注意要取消application.yml中指定sql的配置,因为这样重新运行表会重新创建,消失。

## 整合Druid数据源

### 简介

DRUID是阿里巴巴开源平台上一个数据库连接池实现，它结合了C3P0、DBCP、PROXOOL等DB池的优点，同时加入了日志监控，可以很好的监控DB池连接和SQL的执行情况，可以说是针对监控而生的DB连接池(据说是目前最好的连接池)

### 步骤

在pom文件中加入Druid依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.16</version>
</dependency>
```

### 测试

![A5CwF0.png](https://s2.ax1x.com/2019/04/08/A5CwF0.png)

以成功更换

Debug

![A5ChY6.png](https://s2.ax1x.com/2019/04/08/A5ChY6.png)



发现配置未生效，我们需要编写一个类来实现它。

```java
package com.hph.springboot.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return  new DruidDataSource();

    }
}
```



![A5CLTI.png](https://s2.ax1x.com/2019/04/08/A5CLTI.png)

### 监控

在`DruidConfig`中

```java
package com.hph.springboot.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return  new DruidDataSource();
    }

    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();

        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");//默认就是允许所有访问
        initParams.put("deny","192.168.1.110");

        bean.setInitParameters(initParams);
        return bean;
    }


    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");

        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));

        return  bean;
    }
}

```

![A5i3rQ.png](https://s2.ax1x.com/2019/04/08/A5i3rQ.png)

![A5iUP0.png](https://s2.ax1x.com/2019/04/08/A5iUP0.png)

![A5id2T.png](https://s2.ax1x.com/2019/04/08/A5id2T.png)

![A5iwxU.png](https://s2.ax1x.com/2019/04/08/A5iwxU.png)

Web中也可以查看我们JDBC的执行次数。







