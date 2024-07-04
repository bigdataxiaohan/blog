---
title: Spring概述
date: 2019-03-02 21:11:07
tags: Spring
categories: JavaWeb
---

​       Spring是一个开源框架, Spring为简化企业级开发而生，使用Spring，JavaBean就可以实现很多以前要靠EJB才能实现的功能。同样的功能，在EJB中要通过繁琐的配置和复杂的代码才能够实现，而在Spring中却非常的优雅和简洁。   Spring是一个**IOC**(DI)和**AOP**容器框架。有着优良的特性:

| 特性         | 介绍                                                         |
| ------------ | ------------------------------------------------------------ |
| 非侵入式     | 基于Spring开发的应用中的对象可以不依赖于Spring的API          |
| 依赖注入     | DI——Dependency<br/>Injection，反转控制(IOC)最经典的实现。    |
| 面向切面编程 | Aspect<br/>Oriented Programming——AOP                         |
| 容器         | Spring是一个容器，因为它包含并且管理应用对象的生命周期       |
| 组件化       | Spring实现了使用简单的组件配置组合成一个复杂的应用。在 Spring 中可以使用XML和Java注解组合这些对象。 |
| 一站式       | 在IOC和AOP的基础上可以整合各种企业应用的开源框架和优秀的第三方类库（实际上Spring 自身也提供了表述层的SpringMVC和持久层的Spring JDBC）。 |



##    Spring模块

![kqlqrq.png](https://s2.ax1x.com/2019/03/02/kqlqrq.png)

**核心容器（Spring Core）**

　　核心容器提供Spring框架的基本功能。Spring以bean的方式组织和管理Java应用中的各个组件及其关系。Spring使用BeanFactory来产生和管理Bean，它是工厂模式的实现。BeanFactory使用控制反转(IoC)模式将应用的配置和依赖性规范与实际的应用程序代码分开。

**应用上下文（Spring Context）**

　　Spring上下文是一个配置文件，向Spring框架提供上下文信息。Spring上下文包括企业服务，如JNDI、EJB、电子邮件、国际化、校验和调度功能。

**Spring面向切面编程（Spring AOP）**

　　通过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了 Spring框架中。所以，可以很容易地使 Spring框架管理的任何对象支持 AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。

**JDBC和DAO模块（Spring DAO）**

　　JDBC、DAO的抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理，和不同数据库供应商所抛出的错误信息。异常层次结构简化了错误处理，并且极大的降低了需要编写的代码数量，比如打开和关闭链接。

**对象实体映射（Spring ORM）**

　　Spring框架插入了若干个ORM框架，从而提供了ORM对象的关系工具，其中包括了Hibernate、JDO和 IBatis SQL Map等，所有这些都遵从Spring的通用事物和DAO异常层次结构。

**Web模块（Spring Web）**

　　Web上下文模块建立在应用程序上下文模块之上，为基于web的应用程序提供了上下文。所以Spring框架支持与Struts集成，web模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。

**MVC模块（Spring Web MVC）**

　　MVC框架是一个全功能的构建Web应用程序的MVC实现。通过策略接口，MVC框架变成为高度可配置的。MVC容纳了大量视图技术，其中包括JSP、POI等，模型来有JavaBean来构成，存放于m当中，而视图是一个街口，负责实现模型，控制器表示逻辑代码，由c的事情。Spring框架的功能可以用在任何J2EE服务器当中，大多数功能也适用于不受管理的环境。Spring的核心要点就是支持不绑定到特定J2EE服务的可重用业务和数据的访问的对象，毫无疑问这样的对象可以在不同的J2EE环境，独立应用程序和测试环境之间重用。

## 开发工具

可以使用STS  IDEA ECLIPSE 等 个人比较喜欢IDEA 

## 案例

![kq1soT.md.png](https://s2.ax1x.com/2019/03/02/kq1soT.md.png)

![kq1Wl9.png](https://s2.ax1x.com/2019/03/02/kq1Wl9.png)

等待下载

![kq1Iw6.png](https://s2.ax1x.com/2019/03/02/kq1Iw6.png)

下载完后的目录结构IDE可以帮助我们把需要的jar包下载下来

![kq1jOI.png](https://s2.ax1x.com/2019/03/02/kq1jOI.png)

创建一个类结构如下

![kq36Bt.png](https://s2.ax1x.com/2019/03/02/kq36Bt.png)



配置

```xml
    <!-- 使用bean元素定义一个由IOC容器创建的对象 -->
    <!-- class属性指定用于创建bean的全类名 -->
    <!-- id属性指定用于引用bean实例的标识 -->
    <bean id="student" class="com.hph.helloworld.bean.Student">
       <!--使用property子元素为bean属性赋值-->
    <property name="id" value="1001"></property>
    <property name="name" value="Spring"></property>
    <property name="age" value="18"></property>
    </bean>
```

创建Main方法

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;


public class Main {
    public static void main(String[] args) {
        //1.创建IOC容器对象
        ApplicationContext iocContainer =
                new ClassPathXmlApplicationContext("helloworld.xml");
		//2.根据id值获取bean实例对象
        Student student = (Student) iocContainer.getBean("student");
		//3.打印bean
        System.out.println(student);

    }
}
```

运行结果

![kq8k4O.png](https://s2.ax1x.com/2019/03/02/kq8k4O.png)



