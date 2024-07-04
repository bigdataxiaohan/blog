---
title: MyBatis全局配置文件和映射文件
date: 2019-03-18 12:47:06
tags: Mybatis
categories: JavaWeb
---

##  配置文件

MyBatis 的配置文件包含了影响 MyBatis 行为的设置（settings）和属性（properties）信息。

### 配置文件结构

```properties
configuration 配置 
properties 属性
settings 设置
typeAliases 类型命名
typeHandlers 类型处理器
objectFactory 对象工厂
plugins 插件
environments 环境 
	environment 环境变量 
		transactionManager 事务管理器
		dataSource 数据源
databaseIdProvider 数据库厂商标识
mappers 映射器
```

### properties属性

可外部配置且可动态替换的，既可以在典型的Java 属性文件中配置，也可以通过 properties 元素的子元素来配置

```xml
<properties>
     <property name="driver" value="com.mysql.jdbc.Driver" />
     <property name="url" 
             value="jdbc:mysql://58.87.70.124:3306/test_mybatis" />
     <property name="username" value="root" />
     <property name="password" value="123456" />
 </properties>
```

你可以创建一个资源文件，名为jdbc.properties的文件,将四个连接字符串的数据在资源文件中通过键值 对(key=value)的方式放置，不要任何符号，一条占一行.

mybastis-conf.xml

````xml
<!-- 
		properties: 引入外部的属性文件
			resource: 从类路径下引入属性文件 
			url:  引入网络路径或者是磁盘路径下的属性文件
-->
<properties resource="db.properties" ></properties>
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
````

### jdbc.propertis

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://58.87.70.124:3306/test_mybatis
jdbc.username=root
jdbc.password=123456
```

### settings设置

 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。

```xml
<settings>
<setting name="cacheEnabled" value="true"/>
<setting name="lazyLoadingEnabled" value="true"/>
<setting name="multipleResultSetsEnabled" value="true"/>
<setting name="useColumnLabel" value="true"/>
<setting name="useGeneratedKeys" value="false"/>
<setting name="autoMappingBehavior" value="PARTIAL"/>
<setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
<setting name="defaultExecutorType" value="SIMPLE"/>
<setting name="defaultStatementTimeout" value="25"/>
<setting name="defaultFetchSize" value="100"/>
<setting name="safeRowBoundsEnabled" value="false"/>
<setting name="mapUnderscoreToCamelCase" value="false"/>
<setting name="localCacheScope" value="SESSION"/>
<setting name="jdbcTypeForNull" value="OTHER"/>
<setting name="lazyLoadTriggerMethods"
           value="equals,clone,hashCode,toString"/>
</settings>

```

![AmG9eS.png](https://s2.ax1x.com/2019/03/18/AmG9eS.png)

![AmG9eS.png](https://s2.ax1x.com/2019/03/18/AmG9eS.png)

[原图片链接](https://blog.csdn.net/fageweiketang/article/details/80767532)

### typeAliases


类型别名是为 Java 类型设置一个短的名字，可以方便我们引用某个类。

```xml
    <typeAliases>
    	 <typeAlias type="com.hph.mybatis.beans.Employee" alias="employee"/>
    </typeAliases>
```


类很多的情况下，可以批量设置别名这个包下的每一个类创建一个默认的别名，就是简单类名小写

```xml
    <typeAliases>
        <!--  <typeAlias type="com.hph.mybatis.beans.Employee" alias="employee"/> -->
        <package name="com.hph.mybatis.beans"/>
    </typeAliases>
```


MyBatis已经取好的别名

![AmGsSI.png](https://s2.ax1x.com/2019/03/18/AmGsSI.png)

###  environments

- MyBatis可以配置多种环境，比如开发、测试和生产环境需要有不同的配置
- 每种环境使用一个environment标签进行配置并指定唯一标识符
- 可以通过environments标签中的default属性指定一个环境的标识符来快速的切换环境
- environment-指定具体环境
- id：指定当前环境的唯一标识  transactionManager、和dataSource都必须有

```xml
<environments default="oracle">
		<environment id="mysql">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
		 <environment id="oracle">
			<transactionManager type="JDBC"/>	
			<dataSource type="POOLED">
				<property name="driver" value="${orcl.driver}" />
				<property name="url" value="${orcl.url}" />
				<property name="username" value="${orcl.username}" />
				<property name="password" value="${orcl.password}" />
			</dataSource>
		</environment> 
	</environments>
```

### transactionManager

#### type参数

 JDBC：使用JDBC的提交和回滚设置，依赖于从数据源得到的连接来管理事务范围。 JdbcTransactionFactory

MANAGED：不提交或回滚一个连接、让容器来管理事务的整个生命周期（比如JEE应用服务器的上下文）。 ManagedTransactionFactory

自定义：实现TransactionFactory接口，type=全类名/别名

#### dataSource

UNPOOLED：不使用连接池， UnpooledDataSourceFactory

POOLED：使用连接池， PooledDataSourceFactory

JNDI： 在EJB 或应用服务器这类容器中查找指定的数据源

实际开发中我们使用Spring管理数据源，并进行事务控制的配置来覆盖上述配置

### mappers

 在mybatis初始化的时候，mybatis需要引入的Mapper映射文件.

mapper逐个注册SQL映射文件

&nbsp; &nbsp; &nbsp; &nbsp; resource : 引入类路径下的文件 

&nbsp; &nbsp; &nbsp; &nbsp; url :   引入网络路径或者是磁盘路径下的文件

&nbsp; &nbsp; &nbsp; &nbsp; class :    引入Mapper接口.

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 有SQL映射文件 , 要求Mapper接口与 SQL映射文件同名同位置. 

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 没有SQL映射文件 , 使用注解在接口的方法上写SQL语句.

```xml
<mappers>
		<mapper resource="EmployeeMapper.xml" />
		<mapper class="com.hph.mybatis.dao.EmployeeMapper"/>
		<package name="com.atguigu.mybatis.dao"/>
</mappers>
```

```xml
    <mappers>
        <package name="com.hph.mybatis.dao"></package>
    </mappers>
```

#### 注意

如果是使用的IDEA的话  IDEA 默认不扫描src/main/java 目录会出现org.apache.ibatis.binding.BindingException: Invalid bound statement (not found):  [方法解决链接](https://blog.csdn.net/iteye_14811/article/details/82673913)

### MyBatis映射文件

MyBatis的真正强大在于它的映射语句，也是它的魔力所在。由于它的异常强大，映射器的 XML 文件就显得相对简单。如果拿它跟具有相同功能的 JDBC 代码进行对比，你会立即发现省掉了将近 95% 的代码。MyBatis 就是针对SQL 构建的，并且比普通的方法做的更好。

SQL 映射文件有很少的几个顶级元素（按照它们应该被定义的顺序）：

 &nbsp; &nbsp; &nbsp; &nbsp;cache – 给定命名空间的缓存配置。

 &nbsp; &nbsp; &nbsp; &nbsp;cache-ref – 其他命名空间缓存配置的引用。

 &nbsp; &nbsp; &nbsp; &nbsp;resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加对象。

 &nbsp; &nbsp; &nbsp; &nbsp;parameterMap – 已废弃！老式风格的参数映射。内联参数是首选,这个元素可能在将来被移除。

 &nbsp; &nbsp; &nbsp; &nbsp;sql – 可被其他语句引用的可重用语句块。

 &nbsp; &nbsp; &nbsp; &nbsp;insert – 映射插入语句

 &nbsp; &nbsp; &nbsp; &nbsp;update – 映射更新语句

  &nbsp; &nbsp;&nbsp; &nbsp;delete – 映射删除语句

 &nbsp; &nbsp; &nbsp; &nbsp;select – 映射查询语

















