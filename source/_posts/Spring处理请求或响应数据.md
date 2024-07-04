---
title: SpringMVC处理请求或响应数据
date: 2019-03-07 19:53:04
tags: SpringMVC
categories: JavaWeb
---

## 请求处理方法签名

1. Spring MVC 通过分析处理方法的签名，HTTP请求信息绑定到处理方法的相应人参中。
2. Spring MVC 对控制器处理方法签名的限制是很宽松的，几乎可以按喜欢的任何方式对方法进行签名。 
3. 必要时可以对方法及方法入参标注相应的注解（ @PathVariable 、@RequestParam、@RequestHeader 等）、
4. Spring MVC 框架会将 HTTP 请求的信息绑定到相应的方法入参中，并根据方法的返回值类型做出相应的后续处理。

### @RequestParam注解

1. 在处理方法入参处使用 @RequestParam 可以把请求参数传递给请求方法
2. value：参数名
3. required：是否必须。默认为 true, 表示请求参数中必须包含对应的参数，若不存在，将抛出异常
4. defaultValue: 默认值，当没有传递参数时使用该值

```java
/**
 * @RequestParam 注解用于映射请求参数
 *         value 用于映射请求参数名称
 *         required 用于设置请求参数是否必须的
 *         defaultValue 设置默认值，当没有传递参数时使用该值
 */
   
	@RequestMapping("/testRequestParm")
    public String testRequestParm(@RequestParam("username") String username, @RequestParam(value = "age", required = false, defaultValue = "0") int age) {
        System.out.println(username + "," + age);
        return "success";
    }
```

```html
<a href="springmvc/testRequestParm?username=Tom&age=22">testRequestParm</a>
```

### @RequestHeader 注解

1. 使用 @RequestHeader 绑定请求报头的属性值
2. 请求头包含了若干个属性，服务器可据此获知客户端的信息，**通过** **@RequestHeader** **即可将请求头中的属性值绑定到处理方法的入参中** 

```java
   @RequestMapping("/testRequestHeader")
    public String testRequestHeader(@RequestHeader("Accept-Language") String accpetLanguage) {

        System.out.println("accept-Language:" + accpetLanguage);
        return "success";
    }
```

```html
<a href="springmvc/testRequestHeader">Test RequestHeader</a>
```

###  @CookieValue 注解

1. 使用 @CookieValue 绑定请求中的 Cookie 值

2. **@CookieValue** 可让处理方法入参绑定某个 Cookie 值

```java
    @RequestMapping("/testCookieValue")
    public String testCookieValue(@CookieValue("JSESSIONID") String sessionId) {
        System.out.println("sessionid:" + sessionId);
        return "success";
    }
```

```html
<a href="springmvc/testCookieValue">Test CookieValue</a>
```

### 使用POJO作为参数

1. 使用 POJO 对象绑定请求参数值
2. Spring MVC **会按请求参数名和 POJO** **属性名进行自动匹配，自动为该对象填充属性值**。**支持级联属性**。如：dept.deptId、dept.address.tel 等

```java
package com.hph.springmvc.beans;

public class Address {
    private String province;
    private String city;

    @Override
    public String toString() {
        return "Address{" +
                "province='" + province + '\'' +
                ", city='" + city + '\'' +
                '}';
    }

    public String getProvince() {
        return province;
    }

    public void setProvince(String province) {
        this.province = province;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }
}

```

```java
package com.hph.springmvc.beans;

public class User {
    private String username;
    private String password;
    private String email;
    private Integer gender;
    private  Address address;

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", email='" + email + '\'' +
                ", gender=" + gender +
                ", address=" + address +
                '}';
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
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

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }
}
```

 如果中文有乱码，需要配置字符编码过滤器，且配置其他过滤器之前，如（HiddenHttpMethodFilter），否则不起作用。

```xml
    <!-- 配置编码方式过滤器,注意一点:要配置在所有过滤器的前面 -->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
```

### 使用Servlet原生API作为参数

- HttpServletRequest

- HttpServletResponse
- HttpSession
- **java.security.Principal**
- **Locale**
- InputStream
- OutputStream
- Reader
- Writer

```java
    @RequestMapping("/testServletAPI")
    public void testServletAPI(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException { //需要导入servlet-api.jar
        System.out.println("request :" + request);
        System.out.println("response : " + response);
        //转发
        //   request.getRequestDispatcher("/WEB-INF/views/success.jsp").forward(request, response);
        //重定向 将数据写给客户端
      //  response.sendRedirect("http://www.baidu.com");

        response.getWriter().println("Hello World");
    }
```

```html
<a href="springmvc/testServletAPI">Test Servlet API</a>
```

## 处理响应数据

### SpringMVC 输出模型数据概述

 **ModelAndView**: 处理方法返回值类型为 ModelAndView 时, 方法体即可通过该对象添加模型数据

 **Map** **及 Model**: 入参为 org.springframework.ui.Model、org.springframework.ui.ModelMap 或 java.uti.Map 时，处理方法返回时，Map 中的数据会自动添加到模型中。

#### ModelAndView

控制器处理方法的返回值如果为 ModelAndView, 则其既包含视图信息，也包含模型数据信息。

**添加模型数据:**

MoelAndView addObject(String attributeName, Object attributeValue)

ModelAndView addAllObject(Map<String, ?> modelMap)

**设置视图:**

void setView(View view)

void setViewName(String viewName)

#### 处理模型数据之 Map

Spring MVC 在内部使用了一个 org.springframework.ui.Model 接口存储模型数据

具体使用步骤

**Spring MVC** **在调用方法前会创建一个隐含的模型对象作为模型数据的存储容器**。

**如果方法的入参为 Map** **或 Model** **类型**，Spring MVC 会将隐含模型的引用传递给这些入参。

在方法体内，开发者可以通过这个入参对象访问到模型中的所有数据，也可以向模型中添加**新的属性数据**

![ASMOAA.png](https://s2.ax1x.com/2019/03/09/ASMOAA.png)

## 视图解析器

不论控制器返回一个String,ModelAndView,View都会转换为ModelAndView对象，由视图解析器解析视图，然后，进行页面的跳转。 

![ASQSc8.png](https://s2.ax1x.com/2019/03/09/ASQSc8.png)

### 视图和视图解析器

请求处理方法执行完成后，最终返回一个 ModelAndView 对象。对于那些返回 String，View 或 ModeMap 等类型的处理方法，**Spring MVC** **也会在内部将它们装配成一个 ModelAndView** **对象**，它包含了逻辑名和模型对象的视图。

Spring MVC 借助**视图解析器**（**ViewResolver**）得到最终的视图对象（View），最终的视图可以是 JSP ，也可能是 Excel、JFreeChart等各种表现形式的视图。

对于最终究竟采取何种视图对象对模型数据进行渲染，处理器并不关心，处理器工作重点聚焦在生产模型数据的工作上，从而实现 MVC 的充分解耦。

### 视图

**视图**的作用是渲染模型数据，将模型里的数据以某种形式呈现给客户,主要就是完成转发或者是重定向的操作.

为了实现视图模型和具体实现技术的解耦，Spring 在 org.springframework.web.servlet 包中定义了一个高度抽象的 **View** 接口：

![ASQpjS.png](https://s2.ax1x.com/2019/03/09/ASQpjS.png)

**视图对象由视图解析器负责实例化**。由于视图是**无状态**的，所以他们不会有**线程安全**的问题

![ASQCng.png](https://s2.ax1x.com/2019/03/09/ASQCng.png)

### JstlView

若项目中使用了JSTL，则SpringMVC 会自动把视图由InternalResourceView转为 **JstlView** 

1）    若希望直接响应通过 SpringMVC 渲染的页面，可以使用 **mvc:view-controller** 标签实现,不经过Handler直接跳转页面但是会导致RequestMapping映射失效,因此需要加上 annotation-driven的配置

```xml
<mvc:view-controller path="testViewContorller" view-name="success"></mvc:view-controller>
```

## 视图解析器

SpringMVC 为逻辑视图名的解析提供了不同的策略，可以在 Spring WEB 上下文中**配置一种或多种解析策略**，**并指定他们之间的先后顺序**。每一种映射策略对应一个具体的视图解析器实现类。

视图解析器的作用比较单一：将逻辑视图解析为一个具体的视图对象。

所有的视图解析器都必须实现 ViewResolver 接口：

```java
public interface ViewResolver {

	/**
	 * Resolve the given view by name.
	 * <p>Note: To allow for ViewResolver chaining, a ViewResolver should
	 * return {@code null} if a view with the given name is not defined in it.
	 * However, this is not required: Some ViewResolvers will always attempt
	 * to build View objects with the given name, unable to return {@code null}
	 * (rather throwing an exception when View creation failed).
	 * @param viewName name of the view to resolve
	 * @param locale Locale in which to resolve the view.
	 * ViewResolvers that support internationalization should respect this.
	 * @return the View object, or {@code null} if not found
	 * (optional, to allow for ViewResolver chaining)
	 * @throws Exception if the view cannot be resolved
	 * (typically in case of problems creating an actual View object)
	 */
	View resolveViewName(String viewName, Locale locale) throws Exception;

}
```

### 常用的视图解析器实现类

![ASQkAs.png](https://s2.ax1x.com/2019/03/09/ASQkAs.png)

### mvc:view-controller标签

```xml
    <mvc:view-controller path="testViewContorller" view-name="success"></mvc:view-controller>
	<mvc:annotation-driven></mvc:annotation-driven>
```

### 重定向

①   一般情况下，控制器方法返回字符串类型的值会被当成逻辑视图名处理

②   如果返回的字符串中带 **forward:** **或** **redirect:** 前缀时，SpringMVC 会对他们进行特殊处理：将 forward: 和 redirect: 当成指示符，其后的字符串作为 URL 来处理

③   redirect:success.jsp：会完成一个到 success.jsp 的重定向的操作

④   forward:success.jsp：会完成一个到 success.jsp 的转发操作

## 代码实例

### 实例结构

![ASQQHJ.png](https://s2.ax1x.com/2019/03/09/ASQQHJ.png)

### SpringMVCHandler

```java
package com.hph.springmvc.handler;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import java.util.Date;
import java.util.Map;

@Controller
public class SpringMVCHandler {
    /**
     * 重定向
     */
    @RequestMapping("/testRedirectView")
    public String testRedirctView() {

        return "redirect:/ok.jsp";
    }

    /**
     * 视图 View
     */
    @RequestMapping("/testView")
    public String testView() {
        return "success";
    }

    /**
     * Model
     */
    @RequestMapping("/testModel")
    public String testModel(Model model) {
        //模型数据:loginMsg =用户名或者密码错误
        model.addAttribute("loginMsg", "用户名或者密码错误");
        return "success";
    }

    /**
     * Map
     * SpringMVC会把Map中的模型数据存放到request与对象中
     * SpringMVC会在调完请求处理方法后不管方法的返回是什么类型都会处理成一个ModeAndView(参考DispathcherServlet)
     */
    @RequestMapping("/testMap")
    public String testMap(Map<String, Object> map) {
        //模型数据 password=123456
        map.put("password", "123456");
        return "success";
    }


    @RequestMapping("/testModelAndView")
    public ModelAndView testModelAndView() {
        System.out.println("testModelAndView");
        String viewName = "success";
        ModelAndView mv = new ModelAndView(viewName);
        mv.addObject("time", new Date().toString()); //实质上存放到request域中
        return mv;
    }
}

```

### springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

    <!-- 组件扫描 -->
    <context:component-scan base-package="com.hph.springmvc.handler"></context:component-scan>

    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"></property>
        <property name="suffix" value=".jsp"></property>

        <!--配置视图解析器的优先级-->
        <property name="order" value="100"></property>
    </bean>
    <!--不经过Handler直接跳转页面 但是会导致RequestMapping映射失效,因此需要加上 annotation-driven的配置 -->
    <!---->
    <mvc:view-controller path="testViewContorller" view-name="success"></mvc:view-controller>
    <mvc:annotation-driven></mvc:annotation-driven>
</beans>
```

### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>springDispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!--
        load-on-startup:设置DispathcerServlet
            Servlet的创建时机:
                   请求到达以后创建:
                   服务器启动即创建:
        -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springDispatcherServlet</servlet-name>
        <!--任何请求都会进去 对于JSP请求 不会处理-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

### index.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>$Title$</title>
</head>
<body>
<a href="testRedirectView">Test RedirectView</a>
<br>
<a href="testViewContorller">Test ViewController</a>
<br>
<a href="testView">Test View</a>
<br>
<a href="testModel">Test Model</a>
<br>
<!--测试 ModelAndView 作为处理返回结果 -->
<a href="testModelAndView">testModelAndView</a>
<br>
<a href="testMap">testMap</a>
</body>
</html>

```

### ok.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>OK</title>
</head>
<body>
<h1>OK Page</h1>
</body>
</html>
```

### success.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>Success Page</h1>
time:${requestScope.time}
<br/>
password: ${requestScope.password}
<br/>
loginMsg:${requestScope.loginMsg}
</body>
</html>
```

### 测试

![ASQ841.png](https://s2.ax1x.com/2019/03/09/ASQ841.png)

![ASQJ9x.png](https://s2.ax1x.com/2019/03/09/ASQJ9x.png)

![ASQY36.png](https://s2.ax1x.com/2019/03/09/ASQY36.png)

![ASQauD.png](https://s2.ax1x.com/2019/03/09/ASQauD.png)

![ASQdDe.png](https://s2.ax1x.com/2019/03/09/ASQdDe.png)

![ASQdDe.png](https://s2.ax1x.com/2019/03/09/ASQdDe.png)

![ASQwHH.png](https://s2.ax1x.com/2019/03/09/ASQwHH.png)





















