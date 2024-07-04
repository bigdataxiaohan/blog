---
title: Spring声明式事务
date: 2019-03-05 19:28:33
tags: Spring
categories: JavaWeb
---

## 事务概述

在JavaEE企业级开发的应用领域，为了保证数据的**完整性**和**一致性**，必须引入数据库事务的概念，所以事务管理是企业级应用程序开发中必不可少的技术。

事务就是一组由于逻辑上紧密关联而合并成一个整体(工作单元)的多个数据库操作，这些操作**要么都执行**，**要么都不执行**。 

### ACID:

- 原子性(atomicity)：“原子”的本意是**“不可再分”**，事务的原子性表现为一个事务中涉及到的多个操作在逻辑上缺一不可。事务的原子性要求事务中的所有操作要么都执行，要么都不执行。 
- 一致性(consistency)：“一致”指的是数据的一致，具体是指：所有数据都处于**满足业务规则的一致性状态 **。一致性原则要求：一个事务中不管涉及到多少个操作，都必须保证事务执行之前数据是正确的，事务执行之后数据仍然是正确的。如果一个事务在执行的过程中，其中某一个或某几个操作失败了，则必须将其他所有操作撤销，将数据恢复到事务执行之前的状态，这就是**回滚**。
- 隔离性(isolation)：在应用程序实际运行过程中，事务往往是并发执行的，所以很有可能有许多事务同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。隔离性原则要求多个事务在**并发执行过程中不会互相干扰。**
- 持久性(durability)：持久性原则要求事务执行完成后，对数据的修改**永久的保存**下来，不会因各种系统错误或其他意外情况而受到影响。通常情况下，事务对数据的修改应该被写入到持久化存储器中。

## 编程式事务管理

使用原生的JDBC API进行事务管理:

 ①获取数据库连接Connection对象

②取消事务的自动提交

③执行操作

④正常完成操作时手动提交事务

⑤执行失败时回滚事务

⑥关闭相关资源

## 声明式事务

大多数情况下声明式事务比编程式事务管理更好：它将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。

事务管理代码的固定模式作为一种横切关注点，可以通过AOP方法模块化，进而借助Spring AOP框架实现声明式事务管理。

Spring在不同的事务管理API之上定义了一个**抽象层**，通过**配置**的方式使其生效，从而让应用程序开发人员**不必了解事务管理API的底层实现细节，就可以使用Spring的事务管理机制。

Spring既支持编程式事务管理，也支持声明式的事务管理。

## Spring提供的事务管理器

Spring从不同的事务管理API中抽象出了一整套事务管理机制，让事务管理代码从特定的事务技术中独立出来。开发人员通过配置的方式进行事务管理，而不必了解其底层是如何实现的。

Spring的核心事务管理抽象是PlatformTransactionManager。它为事务管理封装了一组独立于技术的方法。无论使用Spring的哪种事务管理策略(编程式或声明式)，事务管理器都是必须的。

事务管理器可以以普通的bean的形式声明在Spring IOC容器中。

## 事务管理器的主要实现

DataSourceTransactionManager：在应用程序中只需要处理一个数据源，而且通过JDBC存取。

JtaTransactionManager：在JavaEE应用服务器上用JTA(Java Transaction API)进行事务管理

HibernateTransactionManager：用Hibernate框架存取数据库

## 前置准备

![kvpSII.png](https://s2.ax1x.com/2019/03/06/kvpSII.png)

### Dao准备

```java
package com.hph.spring.thing.annotation.dao;

public interface BookShopDao {
    //根据书号查询的书的价格
    public int findPriceByISbn(String isbn);

    //更新书的库存
    public void updateStock(String isbn);

    //更新用户的月
    public void updateUserAccount(String username, Integer price);

}
```

### Daoimpl
```java
package com.hph.spring.thing.annotation.daoimp;

import com.hph.spring.thing.annotation.dao.BookShopDao;
import com.hph.spring.thing.annotation.exception.BookStockException;
import com.hph.spring.thing.annotation.exception.UserAccountException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class BookShopDaoImpl implements BookShopDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;


    @Override
    public int findPriceByISbn(String isbn) {
        String sql = "select price from book where isbn = ?";

        return jdbcTemplate.queryForObject(sql, Integer.class, isbn);
    }

    @Override
    public void updateStock(String isbn) {
        //判断库存是否足够
        String sql = "select stock from book_stock where isbn = ?";
        Integer stock = jdbcTemplate.queryForObject(sql, Integer.class, isbn);
        if (stock <= 0) {
            throw new BookStockException("库存不足");
        }
        sql = "update book_stock set stock =stock -1 where isbn =?";
        jdbcTemplate.update(sql, isbn);

    }

    @Override
    public void updateUserAccount(String username, Integer price) {
        //判断余额是否足够
        String sql = "select balance from account where username = ?";
        Integer balance = jdbcTemplate.queryForObject(sql, Integer.class, username);
        if (balance < price) {
            throw new UserAccountException("余额不足");
        }
        sql = "update  account set balance = balance - ? where  username = ?";

        jdbcTemplate.update(sql, price, username);
    }
}

```
### 自定义异常

```java

//自定义库存异常
public class BookStockException extends RuntimeException {
    public BookStockException() {
    }

    public BookStockException(String message) {
        super(message);
    }

    public BookStockException(String message, Throwable cause) {
        super(message, cause);
    }

    public BookStockException(Throwable cause) {
        super(cause);
    }

    public BookStockException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

```java
package com.hph.spring.thing.annotation.exception;

public class UserAccountException extends RuntimeException {
    public UserAccountException() {
    }

    public UserAccountException(String message) {
        super(message);
    }

    public UserAccountException(String message, Throwable cause) {
        super(message, cause);
    }

    public UserAccountException(Throwable cause) {
        super(cause);
    }

    public UserAccountException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}

```

### Service层

```java
public interface BookShopService {
    public void buyBook(String username, String isbn);

}
```

```java
package com.hph.spring.thing.annotation.service;

import com.hph.spring.thing.annotation.dao.BookShopDao;
import com.hph.spring.thing.annotation.exception.UserAccountException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;


@Transactional   //当前类中所有的方法都起作用
@Service
public class BookShopServiceImpl implements BookShopService {
    @Autowired
    private BookShopDao bookShopDao;

    @Transactional(propagation = Propagation.REQUIRES_NEW,isolation = Isolation.READ_COMMITTED,noRollbackFor = {UserAccountException.class})//只对当前的方法起作用
    public void buyBook(String username, String isbn) {
        Integer price = bookShopDao.findPriceByISbn(isbn);
        bookShopDao.updateStock(isbn);
        bookShopDao.updateUserAccount(username, price);
    }
}
```

```java
package com.hph.spring.thing.annotation.service;

import java.util.List;

public interface Cashier {
    public void checkOut(String username, List<String> isbn);
}
```

```java
package com.hph.spring.thing.annotation.service;

import java.util.List;

public interface Cashier {
    public void checkOut(String username, List<String> isbn);
}
```

```java
package com.hph.spring.thing.annotation.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class CashierImpl implements Cashier {
    @Autowired
    private BookShopService bookShopService;

    public void checkOut(String username, List<String> isbns) {
        for (String isbn : isbns) {
            bookShopService.buyBook(username, isbn);
        }
    }
}
```

### Test层

```java
package com.hph.spring.thing.annotation.test;

import com.hph.spring.thing.annotation.dao.BookShopDao;
import com.hph.spring.thing.annotation.service.BookShopService;
import com.hph.spring.thing.annotation.service.Cashier;
import com.hph.spring.thing.annotation.service.CashierImpl;
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.util.ArrayList;
import java.util.List;

public class TestTransaction {

    private BookShopDao bookShopDao;
    private BookShopService bookShopService;
    private Cashier cashier;

    @Before
    public void init() {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("spring-thing.xml");
        bookShopDao = ctx.getBean("bookShopDaoImpl", BookShopDao.class);
        bookShopService = ctx.getBean("bookShopServiceImpl", BookShopService.class);
        System.out.println(bookShopService.getClass().getName());
        cashier = ctx.getBean("cashierImpl", CashierImpl.class);

    }

    @Test
    public void testThing() {
        bookShopService.buyBook("Tom", "1001");
    }

    @Test
    public void testCheckOut() {
        List<String> isbns = new ArrayList<>();
        isbns.add("1001");
        isbns.add("1002");

        cashier.checkOut("Tom", isbns);
    }
}
```

## 数据库表

```sql
CREATE TABLE book (
  isbn VARCHAR (50) PRIMARY KEY,
  book_name VARCHAR (100),
  price INT
) ;

CREATE TABLE book_stock (
  isbn VARCHAR (50) PRIMARY KEY,
  stock INT,
) ;

CREATE TABLE account (
  username VARCHAR (50) PRIMARY KEY,
  balance INT,
) ;

INSERT INTO account (`username`,`balance`) VALUES ('Tom',300);

INSERT INTO book (`isbn`,`book_name`,`price`) VALUES ('1001','BigData',100);
INSERT INTO book (`isbn`,`book_name`,`price`) VALUES ('1002',Java,70);

INSERT INTO book_stock (`isbn`,`stock`) VALUES ('1002',10);
```

## 配置文件准备

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd">
    <!--扫描基包-->
    <context:component-scan base-package="com.hph.spring.thing.annotation"></context:component-scan>
    <!--数据源配置-->
    <context:property-placeholder location="classpath:db.properties"></context:property-placeholder>
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

    <!--NameParameterJdbcTemplate-->
    <bean id="namedParameterJdbcTemplate" class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
        <constructor-arg ref="dataSource"></constructor-arg>
    </bean>
    <!--事务管理器-->
    <bean id="dataSourceTransactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!-- 开启事务注解
        transaction-manager 用来指定事务管理器， 如果事务管理器的id值 是 transactionManager，
                                                       可以省略不进行指定。
    -->
    <tx:annotation-driven transaction-manager="dataSourceTransactionManager"/>

</beans>
```

## 事务的传播行为

事务的传播行为在数据库种不存在,是Spring在TransactionDefinition接口中规定了7种类型的*事务传播行为*,当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

![kv8UAA.png](https://s2.ax1x.com/2019/03/06/kv8UAA.png)

事务传播属性可以在@Transactional注解的propagation属性中定义。

### 测试关系

![kvGCHH.png](https://s2.ax1x.com/2019/03/06/kvGCHH.png)



### REQUIRED传播行为

当bookService的purchase()方法被另一个事务方法checkout()调用时，它默认会在现有的事务内运行。这个默认的传播行为就是REQUIRED。因此在checkout()方法的开始和终止边界内只有一个事务。这个事务只在checkout()方法结束的时候被提交，结果用户一本书都买不了。

![kvGYvT.png](https://s2.ax1x.com/2019/03/06/kvGYvT.png)



###  REQUIRES_NEW传播行为

表示该方法必须启动一个新事务，并在自己的事务内运行。如果有事务在运行，就应该先挂起它。

![kvGwVJ.png](https://s2.ax1x.com/2019/03/06/kvGwVJ.png)



## 事务的隔离级别

### 问题

​         假设现在有两个事务：Transaction01和Transaction02并发执行。

 脏读：

​        ①Transaction01将某条记录的AGE值从20修改为30。

​         ②Transaction02读取了Transaction01更新后的值：30。

​         ③Transaction01回滚，AGE值恢复到了20。

​         ④Transaction02读取到的30就是一个无效的值。

​    不可重复读：

​         ①Transaction01读取了AGE值为20。

​         ②Transaction02将AGE值修改为30。

​         ③Transaction01再次读取AGE值为30，和第一次读取不一致。

​    幻读：

​         ①Transaction01读取了STUDENT表中的一部分数据。

​         ②Transaction02向STUDENT表中插入了新的行。

​         ③Transaction01读取了STUDENT表时，多出了一些行。

### 隔离级别

数据库系统必须具有隔离并发运行各个事务的能力，使它们不会相互影响，避免各种并发问题。**一个事务与其他事务隔离的程度称为隔离级别**。SQL标准中规定了多种事务隔离级别，不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，但并发性越弱。

 **读未提交**：READ UNCOMMITTED 	允许Transaction01读取Transaction02未提交的修改。

 **读已提交**：READ COMMITTED	      要求Transaction01只能读取Transaction02已提交的修改。

**可重复读**：REPEATABLE READ	      确保Transaction01可以多次从一个字段中读取到相同的值，即Transaction01执行期间禁止其它事务对这个字段进行更新。

**串行化**：SERIALIZABLE		            确保Transaction01可以多次从一个表中读取到相同的行，在Transaction01执行期间，禁止其它事务对这个表进行添加、更新、删除操作。可以避免任何并发问题，但性能十分低下。



|                  | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| READ UNCOMMITTED | 有   | 有         | 有   |
| READ COMMITTED   | 无   | 有         | 有   |
| REPEATABLE READ  | 无   | 无         | 有   |
| SERIALIZABLE     | 无   | 无         | 无   |

|                  | Oracle  | MySQL   |
| ---------------- | ------- | ------- |
| READ UNCOMMITTED | ×       | √       |
| READ COMMITTED   | √(默认) | √       |
| REPEATABLE READ  | ×       | √(默认) |
| SERIALIZABLE     | √       | √       |

### 在Spring中指定事务隔离级别

用@Transactional注解声明式地管理事务时可以在@Transactional的isolation属性中设置隔离级别

### 触发事务回滚的异常

默认情况：捕获到RuntimeException或Error时回滚，而捕获到编译时异常不回滚。

#### 设置注解

​     rollbackFor属性：指定遇到时必须进行回滚的异常类型，可以为多个

​      noRollbackFor属性：指定遇到时不回滚的异常类型，可以为多个

```java
    @Transactional(propagation = Propagation.REQUIRES_NEW,isolation = Isolation.READ_COMMITTED,noRollbackFor = {UserAccountException.class})//只对当前的方法起作用
    public void buyBook(String username, String isbn) {
        Integer price = bookShopDao.findPriceByISbn(isbn);
        bookShopDao.updateStock(isbn);
        bookShopDao.updateUserAccount(username, price);
    }
}

```

### 事务的超时和只读属性

由于事务可以在行和表上获得锁，因此长事务会占用资源，并对整体性能产生影响。

如果一个事务只读取数据但不做修改，数据库引擎可以对这个事务进行优化。超时事务属性：事务在强制回滚之前可以保持多久。这样可以防止长期运行的事务占用资源。只读事务属性: 表示这个事务只读取数据但不更新数据, 这样可以帮助数据库引擎优化事务。

```java

    @Transactional(propagation = Propagation.REQUIRES_NEW,isolation = Isolation.READ_COMMITTED,noRollbackFor = {UserAccountException.class},readOnly = true,timeout = 30)//只对当前的方法起作用
    public void buyBook(String username, String isbn) {
        Integer price = bookShopDao.findPriceByISbn(isbn);
        bookShopDao.updateStock(isbn);
        bookShopDao.updateUserAccount(username, price);
    }
```

## 参考资料

尚硅谷Spring相关课程和文档资料

