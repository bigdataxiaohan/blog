---
title: SpringBoot之Thymeleaf
date: 2019-03-29 20:59:03
tags:  Thymeleaf
categories: SpringBoot 
---

## 简介

Thymeleaf是用于Web和独立环境的现代服务器端Java模板引擎。

Thymeleaf的主要目标是将优雅的自然模板带到您的开发工作流程中—HTML能够在浏览器中正确显示，并且可以作为静态原型，从而在开发团队中实现更强大的协作。Thymeleaf能够处理HTML，XML，JavaScript，CSS甚至纯文本。

Thymeleaf的主要目标是提供一个优雅和高度可维护的创建模板的方式。 为了实现这一点，它建立在自然模板的概念之上，以不影响模板作为设计原型的方式将其逻辑注入到模板文件中。 这改善了设计沟通，弥合了前端设计和开发人员之间的理解偏差。

Thymeleaf也是从一开始就设计(特别是HTML5)允许创建完全验证的模板。

![ABng4H.png](https://s2.ax1x.com/2019/03/29/ABng4H.png)

## 处理模板

- HTML
- XML
- TEXT
- JAVASCRIPT
- CSS
- RAW

有两种标记模板模式(HTML和XML)，三种文本模板模式(TEXT，JAVASCRIPT和CSS)和一种无操作模板模式(RAW)。

HTML模板模式将允许任何类型的HTML输入，包括HTML5，HTML4和XHTML。 将不会执行验证或格式检查，并且在输出中尽可能地遵守模板代码/结构。

XML模板模式将允许XML输入。 在这种情况下，代码应该是格式良好的 - 没有未封闭的标签，没有未加引号的属性等等，如果发现格式错误，解析器将会抛出异常。 请注意，将不会执行验证(针对DTD或XML模式)。

TEXT模板模式将允许对非标记性质的模板使用特殊语法。 这种模板的例子可能是文本电子邮件或模板文档。 请注意，HTML或XML模板也可以作为TEXT处理，在这种情况下，它们不会被解析为标记，而每个标记，DOCTYPE，注释等都将被视为纯文本。

JAVASCRIPT模板模式将允许处理Thymeleaf应用程序中的JavaScript文件。这意味着能够像在HTML文件中一样使用JavaScript文件中的模型数据，但是使用特定于JavaScript的集成(例如专门转义或自然脚本)。 JAVASCRIPT模板模式被认为是文本模式，因此使用与TEXT模板模式相同的特殊语法。

CSS模板模式将允许处理Thymeleaf应用程序中涉及的CSS文件。类似于JAVASCRIPT模式，CSS模板模式也是文本模式，并使用TEXT模板模式中的特殊处理语法。

RAW模板模式根本不会处理模板。它意味着用于将未触及的资源(文件，URL响应等)插入正在处理的模板中。例如，可以将HTML格式的外部非受控资源包含在应用程序模板中，从而安全地知道这些资源可能包含的任何Thymeleaf代码都不会被执行。

## 规则

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");

	private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
    //只要我们把HTML页面放在classpath:/templates/，thymeleaf就能自动渲染；
```

## 使用

引入thymeleaf的命名空间

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>成功！</h1>
    <!--th:text 将div里面的文本内容设置为 -->
    <div th:text="${hello}">这是显示欢迎信息</div>
</body>
</html>
```

![ABu6s0.png](https://s2.ax1x.com/2019/03/29/ABu6s0.png)

未经渲染访问

![ABuhi4.png](https://s2.ax1x.com/2019/03/29/ABuhi4.png)

如果使用了模板引擎则使用了后端的数据，前后端更加合用。

## 语法规则

th:text；改变当前元素里面的文本内容；

th：任意html属性；来替换原生属性的值

![ABKkTS.png](https://s2.ax1x.com/2019/03/29/ABKkTS.png)

```properties
Simple expressions:（表达式语法）
    Variable Expressions: ${...}：获取变量值；OGNL；
    		1）、获取对象的属性、调用方法
    		2）、使用内置的基本对象：
    			#ctx : the context object.
    			#vars: the context variables.
                #locale : the context locale.
                #request : (only in Web Contexts) the HttpServletRequest object.
                #response : (only in Web Contexts) the HttpServletResponse object.
                #session : (only in Web Contexts) the HttpSession object.
                #servletContext : (only in Web Contexts) the ServletContext object.
                
                ${session.foo}
            3）、内置的一些工具对象：
#execInfo : information about the template being processed.
#messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
#uris : methods for escaping parts of URLs/URIs
#conversions : methods for executing the configured conversion service (if any).
#dates : methods for java.util.Date objects: formatting, component extraction, etc.
#calendars : analogous to #dates , but for java.util.Calendar objects.
#numbers : methods for formatting numeric objects.
#strings : methods for String objects: contains, startsWith, prepending/appending, etc.
#objects : methods for objects in general.
#bools : methods for boolean evaluation.
#arrays : methods for arrays.
#lists : methods for lists.
#sets : methods for sets.
#maps : methods for maps.
#aggregates : methods for creating aggregates on arrays or collections.
#ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

    Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
    	补充：配合 th:object="${session.user}：
   <div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
    </div>
    
    Message Expressions: #{...}：获取国际化内容
    Link URL Expressions: @{...}：定义URL；
    		@{/order/process(execId=${execId},execType='FAST')}
    Fragment Expressions: ~{...}：片段引用表达式
    		<div th:insert="~{commons :: main}">...</div>
    		
Literals（字面量）
      Text literals: 'one text' , 'Another one!' ,…
      Number literals: 0 , 34 , 3.0 , 12.3 ,…
      Boolean literals: true , false
      Null literal: null
      Literal tokens: one , sometext , main ,…
Text operations:（文本操作）
    String concatenation: +
    Literal substitutions: |The name is ${name}|
Arithmetic operations:（数学运算）
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
Boolean operations:（布尔运算）
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
Comparisons and equality:（比较运算）
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
Conditional operators:条件运算（三元运算符）
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
Special tokens:
    No-Operation: _ 
```

参考文档： http://note.youdao.com/noteshare?id=1cce0eaf21c8f05d667151541cabfc03&sub=75C83C680CC94CF2A73B7D01407A229B

## Demo

```java
package com.hph.springboot.controller;


import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.Arrays;
import java.util.Map;

@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello() {
        return "Hello SpringBoot";
    }

    @RequestMapping("/success")
    public String success(Map<String,Object> map) {
        map.put("hello","<h1>你好 thymeleaf</h1>");
        map.put("users", Arrays.asList("张三","李四","王五","赵六"));
        return "success";
    }
}
```

success.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>成功！</h1>
<!--th:text 将div里面的文本内容设置为 -->
<div id="div01" class="myDiv" th:id="${hello}" th:class="${hello}" th:text="${hello}">这是显示欢迎信息</div>
<hr/>
<div th:text="${hello}"></div>
<div th:utext="${hello}"></div>
<hr/>

<!-- th:each每次遍历都会生成当前这个标签： 3个h4 -->
<h4 th:text="${user}"  th:each="user:${users}"></h4>
<hr/>
<h4>
    <span th:each="user:${users}"> [[${user}]] </span>
</h4>
<form th:action="@{/upload}" method="post" enctype="multipart/form-data">
    <input type="file" name="file">
    <input type="submit"/>
</form>

</body>
</html>
```

![ABQmq0.png](https://s2.ax1x.com/2019/03/29/ABQmq0.png)





