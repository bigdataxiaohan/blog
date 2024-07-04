---
title: Spring IOC容器和Bean的配置
date: 2019-03-02 22:09:10
tags: Spring
categories: JavaWeb
---

##  IOC和DI

###  IOC(Inversion of Control)：反转控制

在应用程序中的组件需要获取资源时，传统的方式是组件主动的从容器中获取所需要的资源，在这样的模式下开发人员往往需要知道在具体容器中特定资源的获取方式，增加了学习成本，同时降低了开发效率。

反转控制的思想完全颠覆了应用程序组件获取资源的传统方式：反转了资源的获取方向——改由容器主动的将资源推送给需要的组件，开发人员不需要知道容器是如何创建资源对象的，只需要提供接收资源的方式即可，极大的降低了学习成本，提高了开发的效率。这种行为也称为查找的**被动形式**。

###  DI(Dependency Injection)：依赖注入

IOC的另一种表述方式：即组件以一些预先定义好的方式(例如：setter 方法)接受来自于容器的资源注入。相对于IOC而言，这种表述更直接。



###  IOC容器在Spring中的实现

在通过IOC容器读取Bean的实例之前，需要先将IOC容器本身实例化。Spring提供了IOC容器的两种实现方式:

BeanFactory：IOC容器的基本实现，是Spring内部的基础设施，是面向Spring本身的，不是提供给开发人员使用的。

ApplicationContext：BeanFactory的子接口，提供了更多高级特性。面向Spring的使用者，几乎所有场合都使用ApplicationContext而不是底层的BeanFactory

### ApplicationContext的主要实现类

 ClassPathXmlApplicationContext：对应类路径下的XML格式的配置文件

FileSystemXmlApplicationContext：对应文件系统中的XML格式的配置文件

在初始化时就创建单例的bean，也可以通过配置的方式指定创建的Bean是多实例的。

###  WebApplicationContext

专门为WEB应用而准备的，它允许从相对于WEB根目录的路径中完成初始化工作

## 通过类型获取bean

从IOC容器中获取bean时，除了通过id值获取，还可以通过bean的类型获取。但如果同一个类型的bean在XML文件中配置了多个，则获取时会抛出异常，所以同一个类型的bean在容器中必须是唯一的。

```java
Student student = iocContainer.getBean(Student.class);
```

可以使用另外一个重载的方法，同时指定bean的id值和类型

```java
Student student = iocContainer.getBean("student",Student.class);
```



## 给bean的属性赋值

###  依赖注入的方式

使用的是set后面的名称忽略set后面首字母的大写

![kqtI0A.png](https://s2.ax1x.com/2019/03/02/kqtI0A.png) 

#### Spring自动匹配合适的构造器

```xml
<bean id="book" class="com.hph.helloworld.bean.Book">
    <constructor-arg value="10001"></constructor-arg>
    <constructor-arg value="Spring"></constructor-arg>
    <constructor-arg value="Rod Johnson"></constructor-arg>
    <constructor-arg value="2019.3"></constructor-arg>
</bean>
```

#### 通过索引值指定参数位置

```xml
    <bean id="book1" class="com.hph.helloworld.bean.Book">
        <constructor-arg value="10002" index="0"></constructor-arg>
        <constructor-arg value="Rod Johnson" index="2"></constructor-arg>
        <constructor-arg value="Spring" index="1"></constructor-arg>
        <constructor-arg value="2019.32" index="3"></constructor-arg>
    </bean>
```

​     

#### 通过类型区分重载的构造器

```xml
    <bean id="book2" class="com.hph.helloworld.bean.Book">
        <constructor-arg value="10003" index="0" type="java.lang.Integer"></constructor-arg>
        <constructor-arg value="2019.3223" index="3" type="java.lang.Double"></constructor-arg>
        <constructor-arg value="Spring" index="1"></constructor-arg>
        <constructor-arg value="Rod Johnson" index="2"></constructor-arg>
    </bean>
```

创建Main方法

```java
 public static void main(String[] args) {
        ApplicationContext iocContainer =
                new ClassPathXmlApplicationContext("helloworld.xml");

        Book book = iocContainer.getBean("book", Book.class);
        System.out.println(book);
        Book book1 = iocContainer.getBean("book1", Book.class);
        System.out.println(book1);
        Book book2 = iocContainer.getBean("book2", Book.class);
        System.out.println(book2);
    }
```

结果

![kqNcHs.png](https://s2.ax1x.com/2019/03/02/kqNcHs.png)

### p名称空间

为了简化XML文件的配置，越来越多的XML文件采用属性而非子元素配置信息。Spring从2.5版本开始引入了一个新的p命名空间，可以通过&lt;&lt;bean&gt;&gt;元素属性的方式配置Bean的属性。使用p命名空间后，基于XML的配置方式将进一步简化。

```xml
    <bean
        id="bookp"
        class="com.hph.helloworld.bean.Book"
        p:id="10004" p:bookName="spring_p" p:author="Rod Johnson" p:price="2019.33"
    ></bean>
```

### 可以使用的值

- 可以使用字符串表示的值，可以通过value属性或value子节点的方式指定

- 基本数据类型及其封装类、String等类型都可以采取字面值注入的方式

- 若字面值中包含特殊字符，可以使用<![CDATA[]]>把字面值包裹起来

#### null值

```xml

    <bean class="com.hph.helloworld.bean.Book" id="bookNull">
        <property name="id" value="100004"></property>
        <property name="bookName">
            <null></null>
        </property>
        <property name="author" value="nullAuthhor"></property>
        <property name="price" value="50"></property>
    </bean>

```

#### 级联属性赋值

![kqWrFK.png](https://s2.ax1x.com/2019/03/03/kqWrFK.png)

创建两个类结构如上

配置xml文件

```xml
    <bean id="car" class="com.hph.helloworld.bean.Car">
        <property name="brand" value="奥迪"></property>
        <property name="price" value="300000"></property>
        <property name="speed" value="180"></property>
    </bean>
    <bean id="person" class="com.hph.helloworld.bean.Person">
        <property name="name" value="张三"></property>
        <property name="age" value="35"></property>
        <!--设置级联属性-->
        <property name="car" ref="car"></property>
    </bean>
```

#### 外部已声明的bean

```xml
  	<!-- 外部已声明的bean-->
    <bean id="person1" class="com.hph.helloworld.bean.Person">
        <property name="car" ref="car"></property>
    </bean>
```

Bean就是一个对象

```java
    public static void main(String[] args) {
        ApplicationContext iocContainer =
                new ClassPathXmlApplicationContext("helloworld.xml");

        Person person1 = iocContainer.getBean("person1", Person.class);
        System.out.println(person1);
    }
```

![kqfnfO.png](https://s2.ax1x.com/2019/03/03/kqfnfO.png)

#### 内部bean

bean实例仅仅给一个特定的属性使用时，可以将其声明为内部bean。内部bean声明直接包含在&lt;property&gt;或&lt;constructor-arg&gt;元素里，不需要设置任何id或name属性内部bean不能使用在任何其他地方

## 集合属性

### List

Spring中可以通过一组内置的XML标签来配置集合属性，例如：&lt;list&gt;，&lt;set&gt;或&lt;map&gt;。

配置java.util.List类型的属性，需要指定&lt;list&gt;标签，在标签里包含一些元素。这些标签   可以通过&lt;value&gt;指定简单的常量值，通过&lt;ref&gt;指定对其他Bean的引用。通过&lt;bean&gt;   指定内置bean定义。通过&lt;null/&gt;指定空元素。甚至可以内嵌其他集合。数组的定义和List一样，都使用&lt;list&gt;元素。

 配置java.util.Set需要使用&lt;set&gt;标签，定义的方法与List一样。

```xml
 <!--数组和List-->
    <bean id="booktype" class="com.hph.helloworld.bean.BookList">
        <property name="bookType">
            <!--字面量为值的List集合-->
            <list>
                <value>计算机</value>
                <value>历史</value>
            </list>
        </property>
        <property name="book" ref="book"></property>
    </bean>
```



### Map

​         Java.util.Map通过&lt;map&gt;标签定义，&lt;map&gt标签里可以使用多个&lt;entry&gt;作为子标签。每个条目包含一个键和一个值。  必须在&lt;key&gt;标签里定义键。因为键和值的类型没有限制，所以可以自由地为它们指定&lt;value&gt、&lt;ref&gt;、&lt;bean&gt或&lt;null/&gt;元素。​可以将Map的键和值作为&lt;entry&gt;的属性定义：简单常量使用key和value来定义；bean引用通过key-ref和value-ref属性定义。

```xml
    <bean id="bookMap" class="com.hph.helloworld.bean.BookList">
        <property name="bookInfo">
            <map>
                <entry value="38.8">
                    <key>
                        <value>机械工鞋出版社</value>
                    </key>
                </entry>
            </map>
        </property>
    </bean>
```

### 集合类型的bean

​         如果只能将集合对象配置在某个bean内部，则这个集合的配置将不能重用。我们需要将集合bean的配置拿到外面，供其他bean引用。配置集合类型的bean需要引入util名称空间

```xml
    <util:list id="booklist">
        <ref bean="book"></ref>
        <ref bean="book1"></ref>
        <ref bean="book2"></ref>
    </util:list>

    <util:list id="typeList">
        <value>编程</value>
        <value>极客时间</value>
        <value>历史</value>
    </util:list>
```

## FactoryBean

Spring中有两种类型的bean，一种是普通bean，另一种是工厂bean，即FactoryBean。

​         工厂bean跟普通bean不同，其返回的对象不是指定类的一个实例，其返回的是该工厂bean的getObject方法所返回的对象。工厂bean必须实现org.springframework.beans.factory.FactoryBean接口。

![kqLsYj.png](https://s2.ax1x.com/2019/03/03/kqLsYj.png)

```java
import org.springframework.beans.factory.FactoryBean;

public class BookFactory implements FactoryBean<BookFactory> {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }


    @Override
    public BookFactory getObject() throws Exception {
        BookFactory bean = new BookFactory();
        bean.setName("工厂模式");
        return bean;
    }
//返回类型
    @Override
    public Class<?> getObjectType() {
        return BookFactory.class;
    }
    //返回由FactoryBean创建的bean实例，如果isSingleton()返回true，则该实例会放到Spring容器中单实例缓存池中。
    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

XML文件配置

```xml
<bean id="bookproduct" class="com.hph.helloworld.bean.BookFactory">
        <property name="name" value="工厂模式"></property>
</bean>
```

##  配置信息的继承

Spring允许继承bean的配置，被继承的bean称为父bean。继承这个父bean的bean称为子bean子bean从父bean中继承配置，包括bean的属性配置子bean也可以覆盖从父bean继承过来的配置

```xml
 <bean id="dept" class="com.hph.helloworld.bean.Department">
        <property name="deptId" value="10010"></property>
        <property name="deptName" value="Web"></property>
    </bean>
    <bean id="emp01" class="com.hph.helloworld.bean.Employee">
        <property name="empId" value="1001"></property>
        <property name="empName" value="Jerry"></property>
        <property name="age" value="20"></property>
    </bean>
    <!-- 以emp01作为父bean，继承后可以省略公共属性值的配置 -->
    <bean id="emp02" parent="emp01">
        <property name="empId" value="1002"/>
        <property name="empName" value="Jerry"/>
        <property name="age" value="25"/>
    </bean>
```

**注意:**  父bean可以作为配置模板，也可以作为bean实例。若只想把父bean作为模板，可以设置&lt;bean&gt;的abstract 属性为true，这样Spring将不会实例化这个bean如果一个bean的class属性没有指定，则必须是抽象bean 并不是&lt;bean&gt;元素里的所有属性都会被继承。比如：autowire，abstract等。也可以忽略父bean的class属性，让子bean指定自己的类，而共享相同的属性配置。    但此时abstract必须设为true。

## bean的作用域(重要)

​         在Spring中，可以在&lt;bean&gt;元素的scope属性里设置bean的作用域，以决定这个bean是单实例的还是多实例的。

​         默认情况下，Spring只为每个在IOC容器里声明的bean创建唯一一个实例，整个IOC容器范围内都能共享该实例：所有后续的getBean()调用和bean引用都将返回这个唯一的bean实例。该作用域被称为singleton，它是所有bean的默认作用域。

![kLpARe.png](https://s2.ax1x.com/2019/03/03/kLpARe.png)

​         当bean的作用域为单例时，Spring会在IOC容器对象创建时就创建bean的对象实例。而当bean的作用域为prototype时，IOC容器在获取bean的实例时创建bean的实例对象。

## bean的生命周期

Spring IOC容器可以管理bean的生命周期，Spring允许在bean生命周期内特定的时间点执行指定的任务。

Spring IOC容器对bean的生命周期进行管理的过程：

​         1.通过构造器或工厂方法创建bean实例

​         2. 为bean的属性设置值和对其他bean的引用

​         3.调用bean的初始化方法

​         4.bean可以使用了

​         6.当容器关闭时，调用bean的销毁方法

 在配置bean时，通过init-method和destroy-method 属性为bean指定初始化和销毁方法

 bean的后置处理器:

​     bean后置处理器允许在调用**初始化方法前后**对bean进行额外的处理

​     bean后置处理器对IOC容器里的所有bean实例逐一处理，而非单一实例。其典型应用是：检查bean属性的正确性或根据特定的标准更改bean的属性。

​    bean后置处理器时需要实现接口：

org.springframework.beans.factory.config.BeanPostProcessor。在初始化方法被调用前后，Spring将把每个bean实例分别传递给上述接口的以下两个方法：

postProcessBeforeInitialization(Object, String)

postProcessAfterInitialization(Object, String)

 添加bean后置处理器后bean的生命周期

​         1.通过构造器或工厂方法**创建**bean**实例**

​         2.为bean的**属性设置值**和对其他bean的引用

​         3.将bean实例传递给bean后置处理器的postProcessBeforeInitialization()方法

​         4.调用bean的**初始化**方法

​         5.将bean实例传递给bean后置处理器的postProcessAfterInitialization()方法

​         6.bean可以使用了

​         7.当容器关闭时调用bean的**销毁方法**

## 引用外部属性文件

​         当bean的配置信息逐渐增多时，查找和修改一些bean的配置信息就变得愈加困难。这时可以将一部分信息提取到bean配置文件的外部，以properties格式的属性文件保存起来，同时在bean的配置文件中引用properties属性文件中的内容，从而实现一部分属性值在发生变化时仅修改properties属性文件即可。这种技术多用于连接数据库的基本信息的配置。

jdbc.properties

```xml
jdbc.username=root
jdbc.password=123456
jdbc.url=jdbc:mysql://localhost:3306/bigdata
jdbc.driver=com.mysql.jdbc.Driver
```

XML文件配置

```xml
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="driverClass" value="${jdbc.driver}"/>
    </bean>
```

```java
import org.springframework.context.support.ClassPathXmlApplicationContext;

import javax.sql.DataSource;
import java.sql.SQLException;

public class TestDataSource {
        public static void main(String[] args) {
            ClassPathXmlApplicationContext iocContainer = new ClassPathXmlApplicationContext("helloworld.xml");
            DataSource ds = iocContainer.getBean("dataSource",DataSource.class);
            System.out.println("ds:"+ds);
            try {
                System.out.println(ds.getConnection());
            } catch (SQLException e) {
                e.printStackTrace();
            }

        }
}
```

**注意: ** 要导入相关的jar包和Mysql的驱动.

##  自动装配

手动装配：以value或ref的方式**明确指定属性值**都是手动装配。

自动装配：根据指定的装配规则，**不需要明确指定**，Spring**自动**将匹配的属性值**注入**bean中。

### 装配模式

根据**类型**自动装配：将类型匹配的bean作为属性注入到另一个bean中。若IOC容器中有多个与目标bean类型一致的bean，Spring将无法判定哪个bean最合适该属性，所以不能执行自动装配

根据**名称**自动装配：必须将目标bean的名称和属性名设置的完全相同

通过构造器自动装配：当bean中存在多个构造器时，此种自动装配方式将会很复杂。不推荐使用。

### 注解配置bean

相对于XML方式而言，通过注解的方式配置bean更加简洁和优雅，而且和MVC组件化开发的理念十分契合，是开发中常用的使用方式(推荐使用)。

### 注解标识组件

| 组件             | 注解        | 功能                                           |
| ---------------- | ----------- | ---------------------------------------------- |
| 普通组件         | @Component  | 标识一个受Spring IOC容器管理的组件             |
| 持久化层组件     | @Repository | 标识一个受Spring IOC容器管理的持久化层组件     |
| 业务逻辑层组件   | @Service    | 标识一个受Spring IOC容器管理的业务逻辑层组件   |
| 表述层控制器组件 | @Controller | 标识一个受Spring IOC容器管理的表述层控制器组件 |


#### 组件命名规则

- 默认情况：使用组件的简单类名首字母小写后得到的字符串作为bean的id
- 用组件注解的value属性指定bean的id

注意:事实上Spring并没有能力识别一个组件到底是不是它所标记的类型，即使将 @Respository注解用在一个表述层控制器组件上面也不会产生任何错误，所以@Respository、@Service、@Controller这几个注解仅仅是为了让开发人员自己明确当前的组件扮演的角色。

## 扫描组件

 组件被上述注解标识后还需要通过Spring进行扫描才能够侦测到。

```xml
<context:component-scan base-package="com.hph.component"></context:component-scan>
```

**base-package**属性指定一个需要扫描的基类包，Spring容器将会扫描这个基类包及其子包中的所有类。

当需要扫描多个包时可以使用逗号分隔。

如果仅希望扫描特定的类而非基包下的所有类，可使用resource-pattern属性过滤特定的类，示例：

```xml
<context:component-scan 
	base-package="com.hph.component" 
	resource-pattern="autowire/*.class"/>
```

包含与排除:<context:include-filter>子节点表示要包含的目标类

注意：通常需要与use-default-filters属性配合使用才能够达到“仅包含某些组件”这样的效果。即：通过将use-default-filters属性设置为false，禁用默认过滤器，然后扫描的就只是include-filter中的规则指定的组件了。&lt;ontext:exclude-filter&gt;子节点表示要排除在外的目标类.component-scan下可以拥有若干个include-filter和exclude-filter子节点。

过滤表达式

| 类别       | 示例                  | 说明                                                         |
| :--------- | --------------------- | ------------------------------------------------------------ |
| annotation | com.hph.XxxAnnotation | 过滤所有标注了XxxAnnotation的类。这个规则根据目标组件是否标注了指定类型的注解进行过滤。 |
| assignable | com.hph.BaseXxx       | 过滤所有BaseXxx类的子类。这个规则根据目标组件是否是指定类型的子类的方式进行过滤。 |
| aspectj    | com.hph.*Service+     | 所有类名是以Service结束的，或这样的类的子类。这个规则根据AspectJ表达式进行过滤。 |
| regex      | com\.hph\.anno\.*     | 所有com.hph.anno包下的类。这个规则根据正则表达式匹配到的类名进行过滤。 |
| custom     | com.hph.XxxTypeFilter | 使用XxxTypeFilter类通过编码的方式自定义过滤规则。该类必须实现org.springframework.core.type.filter.TypeFilter接口 |

##  组件装配

 需求：Controller组件中往往需要用到Service组件的实例，Service组件中往往需要用到       Repository组件的实例。Spring可以通过注解的方式帮我们实现属性的装配。

实现依据：在指定要扫描的包时，&lt;context:component-scan&gt; 元素会自动注册一个bean的后置处     理器：AutowiredAnnotationBeanPostProcessor的实例。该后置处理器可以自动装配标记了**@Autowired**、@Resource或@Inject注解的属性。

###  @Autowired注解

- 根据类型实现自动装配。
- 构造器、普通字段(即使是非public)、一切具有参数的方法都可以应用
- @Autowired注解默认情况下，所有使用@Autowired注解的属性都需要被设置。当Spring找不到匹配的bean装配属性时，会抛出异常。
- 若某一属性允许不被设置，可以设置@Autowired注解的required属性为 false
- 默认情况下，当IOC容器里存在多个类型兼容的bean时，Spring会尝试匹配bean 的id值是否与变量名相同，如果相同则进行装配。如果bean的id值不相同，通过类型的自动装配将无法工作。此时可以在@Qualifier注解里提供bean的名称。Spring    甚至允许在方法的形参上标注@Qualifiter注解以指定注入bean的名称。
- @Autowired注解也可以应用在数组类型的属性上，此时Spring将会把所有匹配的bean进行自动装配。
- @Autowired注解也可以应用在集合属性上，此时Spring读取该集合的类型信息，然后自动装配所有与之兼容的bean。
- @Autowired注解用在java.util.Map上时，若该Map的键值为String，那么 Spring将自动装配与值类型兼容的bean作为值，并以bean的id值作为键。

### @Resource

 @Resource注解要求提供一个bean名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为bean的名称。

### @Inject

@Inject和@Autowired注解一样也是按类型注入匹配的bean，但没有reqired属性。





