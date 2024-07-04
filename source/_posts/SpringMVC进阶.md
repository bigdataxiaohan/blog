---

title: SpringMVC处理Json、文件上传、拦截器
date: 2019-03-11 19:16:10
tags: SpringMVC
categories: JavaWeb
---

## 处理JSON

### 链接

http://repo1.maven.org/maven2/com/fasterxml/jackson/core/

### 步骤

编写目标方法，使其返回 JSON 对应的对象或集合

```java
@ResponseBody  //SpringMVC对JSON的支持
@RequestMapping("/testJSON")
public Collection<Employee> testJSON(){
return employeeDao.getAll();
}
```

### idex.jsp

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
 "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
<script type="text/javascript" src="scripts/jquery-1.9.1.min.js"></script>
 
</head>
<body>
<a id="testJSON" href="testJSON">testJSON</a>
</body>
</html>
</body>
</html>
```

### 原理

HttpMessageConverter&lt;T&gt;:
HttpMessageConverter&lt;T&gt;是 Spring3.0 新添加的一个接口，负责将请求信息转换为一个对象（类型为T），将对象（类型为T）输出为响应信息

HttpMessageConverter&lt;T&gt;接口定义的方法：

 Boolean canRead(Class&lt;?&gt; clazz,MediaType mediaType): 指定转换器可以读取的对象类型，即转换器是否可将请求信息转换为 clazz 类型的对象，同时指定支持 MIME 类型(text/html,applaiction/json等)

Boolean canWrite(Class<?> clazz,MediaType mediaType):指定转换器是否可将 clazz 类型的对象写到响应流中，响应流支持的媒体类型在MediaType 中定义。

 List&lt;MediaType&gt;getSupportMediaTypes()：该转换器支持的媒体类型。

T read(Class&lt;? extends &lt;T&gt; clazz,**HttpInputMessage** inputMessage)：将请求信息流转换为 T 类型的对象。

void write(T t,MediaType contnetType,**HttpOutputMessgae** outputMessage):将T类型的对象写到响应流中，同时指定相应的媒体类型为 contentType。

```java
package org.springframework.http;
 
import java.io.IOException;
import java.io.InputStream;
 
public interface HttpInputMessage extends HttpMessage {
 
InputStream getBody() throws IOException;
 
}


package org.springframework.http;
 
import java.io.IOException;
import java.io.OutputStream;
 
public interface HttpOutputMessage extends HttpMessage {
 
OutputStream getBody() throws IOException;
 
}

```



![ACcLDK.png](https://s2.ax1x.com/2019/03/11/ACcLDK.png)

![ACg1bT.png](https://s2.ax1x.com/2019/03/11/ACg1bT.png)

DispatcherServlet 默认装配 RequestMappingHandlerAdapter ，而 RequestMappingHandlerAdapter 默认装配如下 HttpMessageConverter：

![ACgJ54.png](https://s2.ax1x.com/2019/03/11/ACgJ54.png)

 加入 jackson jar 包后， RequestMappingHandlerAdapter  装配的 HttpMessageConverter  如下：

![ACgNG9.png](https://s2.ax1x.com/2019/03/11/ACgNG9.png)

默认情况下数组长度是6个；增加了jackson的包，后多个一个MappingJackson2HttpMessageConverte

## 文件上传

Spring MVC 为文件上传提供了直接的支持，这种支持是通过即插即用的 **MultipartResolver** 实现的。 

Spring 用 **Jakarta Commons FileUpload** 技术实现了一个 MultipartResolver 实现类：**CommonsMultipartResolver**   

Spring MVC上下文中默认没有装配MultipartResovler，因此默认情况下不能处理文件的上传工作，如果想使用 Spring 的文件上传功能，需现在上下文中配置 MultipartResolver

配置MultipartResolver defaultEncoding: 必须和用户JSP 的 pageEncoding 属性一致，以便正确解析表单的内容,为了让 **CommonsMultipartResolver** 正确工作，必须先将 Jakarta Commons FileUpload 及 Jakarta Commons io 的类包添加到类路径下。

### 案例

#### SpringFileHandler

```java
  /**
     * 文件的上传
     * 上传的原理:将本地文件上传到服务器端
     */
    @RequestMapping("/upload")
    public void testUploadFile(@RequestParam("desc") String desc, @RequestParam("uploadFile") MultipartFile uploadFile, HttpSession session) throws IOException {
        //获取到上传文件的名字
        String uploadFileName = uploadFile.getOriginalFilename();
        //获取输入流
        InputStream in = uploadFile.getInputStream();
        //获取服务器端的uploads的真实路径
        ServletContext sc = session.getServletContext();
        String realPath = sc.getRealPath("uploads");
        System.out.println(realPath);
        File targetFile = new File(realPath + "/" + uploadFileName);
        FileOutputStream os = new FileOutputStream(targetFile);
        //写文件
        int i;
        while ((i = in.read()) != -1) {
            os.write(i);
        }
        in.close();
        os.close();
        System.out.println(uploadFileName + "上传成功!");
    }
```

#### springmvc.xml

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
    <context:component-scan base-package="com.hph.springmvc"></context:component-scan>

    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
    <mvc:default-servlet-handler/>
    <mvc:annotation-driven/>
    <!--配置文件上传 必须配置ie-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 保证与上传表单所在的Jsp页面的编码一致. -->
        <property name="defaultEncoding" value="utf-8"></property>
        <property name="maxUploadSize" value="10485760"></property>
    </bean>
</beans>

```

#### web.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID" version="2.5">
  
  <!-- 字符编码过滤器 -->
  <filter>
  	<filter-name>CharacterEncodingFilter</filter-name>
  	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  	<init-param>
  		<param-name>encoding</param-name>
  		<param-value>UTF-8</param-value>
  	</init-param>
  </filter>
  <filter-mapping>
  	<filter-name>CharacterEncodingFilter</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
  
  <!-- REST 过滤器 -->
  <filter>
  	<filter-name>HiddenHttpMethodFilter</filter-name>
  	<filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>
  <filter-mapping>
  	<filter-name>HiddenHttpMethodFilter</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
  
 <!-- 前端控制器 -->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
</web-app>
```

#### index.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>FileTest</title>
</head>
<body>

<form action="upload" method="post" enctype="multipart/form-data">
    上传文件:<input type="file" name="uploadFile">
    <br>
    文件描述:<input type="text" name="desc">
    <br>
    <input type="submit" value="上传">
</form>
</body>
</html>
```

### 测试

![APYOX9.png](https://s2.ax1x.com/2019/03/12/APYOX9.png)

![APYjmR.png](https://s2.ax1x.com/2019/03/12/APYjmR.png)

![APYv01.png](https://s2.ax1x.com/2019/03/12/APYv01.png)

![APYxTx.png](https://s2.ax1x.com/2019/03/12/APYxTx.png)![APtptK.png](https://s2.ax1x.com/2019/03/12/APtptK.png)

成功上传到服务器端

## 多文件上传

### SpringFileHandler

```java
 /**
     * 上传多个文件
     */
    @RequestMapping(value = "/manyFileUpload", method = RequestMethod.POST)
    public String manyFileUpload(@RequestParam("files") MultipartFile[] file, HttpSession session) throws IOException {
        //获取服务器端的uploads的真实路径
        ServletContext sc = session.getServletContext();
        String realPath = sc.getRealPath("uploads");
        for (MultipartFile multipartFile : file) {
            if (!multipartFile.isEmpty()) {
                multipartFile.transferTo(new File(realPath + "/" + multipartFile.getOriginalFilename()));
            }
        }
        return "success";
    }

```

### index.jps

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>FileTest</title>
</head>
<body>

<form action="upload" method="post" enctype="multipart/form-data">
    上传文件:<input type="file" name="uploadFile">
    <br>
    文件描述:<input type="text" name="desc">
    <br>
    <input type="submit" value="上传">
</form>
<br>
<h2>上传多个文件 实例</h2>
<form action="manyFileUpload" method="post"  enctype="multipart/form-data">
    <p>选择文件:<input type="file" name="files"></p>
    <p>选择文件:<input type="file" name="files"></p>
    <p><input type="submit" value="提交"></p>
</form>
</body>
</html>
```

### 测试

![APfIM9.png](https://s2.ax1x.com/2019/03/12/APfIM9.png)

![APhAJS.png](https://s2.ax1x.com/2019/03/12/APhAJS.png)

![APhERg.png](https://s2.ax1x.com/2019/03/12/APhERg.png)

## 文件下载

### 数据准备

![APt9fO.png](https://s2.ax1x.com/2019/03/12/APt9fO.png)

###  SpringFileHandler

```java
 /**
     * 使用HttpMessageConveter完成下载功能
     * 支持 @RequestBody @ResponsBody Httpentity ResponseEntity
     * <p>
     * 下载的原理:   将服务器端的文件已流的形式写到客户端
     * ResponseEntity:将要在的文件数据,以及一些响应信息封装到ResponseEntity对象中,浏览器通过解析发送回去的解析发送回去的响应数据就可以进行下载操作
     */
    @RequestMapping("/download")
    public ResponseEntity<byte[]> testDownload(HttpSession session) throws Exception {
        //将要下载的文件读取成一个z字节数组
        byte[] imgs;

        ServletContext sc = session.getServletContext();
        InputStream in = sc.getResourceAsStream("image/test.png");
        imgs = new byte[in.available()];
        in.read(imgs);
        //响应数据,以及响应头信息封装到ResponseEntity中
        /**
         * 参数:
         *  1.要发送给浏览器的数据
         *  2.设置响应头
         *  3.设置响应码
         */
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Disposition", "attachment;filename=test.png");
        HttpStatus statusCode = HttpStatus.OK;  //200
        ResponseEntity<byte[]> re = new ResponseEntity<byte[]>(imgs, headers, statusCode);

        return re;

    }

```

## 拦截器

Spring MVC也可以使用拦截器对请求进行拦截处理，用户可以自定义拦截器来实现特定的功能，自定义的拦截器必须实现HandlerInterceptor接口

**preHandle**()：这个方法在业务处理器处理请求之前被调用，在该方法中对用户请求 request 进行处理。如果程序员决定该拦截器对请求进行拦截处理后还要调用其他的拦截器，或者是业务处理器去进行处理，则返回true；如果程序员决定不需要再调用其他的组件去处理请求，则返回fals。

**postHandle**()：这个方法在业务处理器处理完请求后，但是DispatcherServlet 向客户端返回响应前被调用，在该方法中对用户请求request进行处理。

**afterCompletion**()：这个方法**在** DispatcherServlet 完全处理完请求后被调用，可以在该方法中进行一些资源清理的操作。

```java
package com.hph.springmvc.interceptor;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 自定义拦截器
 */
@Component
public class MyFirstInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("MyFirstInterceptor preHandler");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("MyFirstInterceptor  postHandle");

    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("MyFirstInterceptor afterCompletion");
    }
}

```

### 案例

####  springmvc.xml

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
    <context:component-scan base-package="com.hph.springmvc"></context:component-scan>

    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
    <mvc:default-servlet-handler/>
    <mvc:annotation-driven/>
    <!--配置文件上传 必须配置ie-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 保证与上传表单所在的Jsp页面的编码一致. -->
        <property name="defaultEncoding" value="utf-8"></property>
        <property name="maxUploadSize" value="10485760"></property>
    </bean>
    <!--配置拦截器-->
    <mvc:interceptors>
        <!--1.拦截所有的请求-->
        <bean class="com.hph.springmvc.interceptor.MyFirstInterceptor"></bean>
    </mvc:interceptors>
</beans>

```

#### MyFirstInterceptor

```java
package com.hph.springmvc.interceptor;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 自定义拦截器
 */
@Component
public class MyFirstInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("MyFirstInterceptor preHandler");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("MyFirstInterceptor  postHandle");

    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("MyFirstInterceptor afterCompletion");
    }
}

```

### 测试

![AP7nL4.png](https://s2.ax1x.com/2019/03/12/AP7nL4.png)

![APIvkt.png](https://s2.ax1x.com/2019/03/12/APIvkt.png)

### 多个拦截器配置

#### MyFirstInterceptor

```java
package com.hph.springmvc.interceptor;

        import org.springframework.stereotype.Component;
        import org.springframework.web.servlet.HandlerInterceptor;
        import org.springframework.web.servlet.ModelAndView;

        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;

/**
 * 自定义拦截器
 */
@Component
public class MyFirstInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("MyFirstInterceptor preHandler");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("MyFirstInterceptor  postHandle");

    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("MyFirstInterceptor afterCompletion");
    }
}

```



#### MySecondInterceptor

```java
package com.hph.springmvc.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
	
public class MySecondInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("[MySecondInterceptor preHandler]");

        return false;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("[MySecondInterceptor  postHandle]");

    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("[MySecondInterceptor afterCompletion]");

    }
}

```



![AiGC24.png](https://s2.ax1x.com/2019/03/12/AiGC24.png)

![AicPAS.png](https://s2.ax1x.com/2019/03/12/AicPAS.png)

![AiGAq1.png](https://s2.ax1x.com/2019/03/12/AiGAq1.png)

![Ai65Ox.png](https://s2.ax1x.com/2019/03/12/Ai65Ox.png)





































