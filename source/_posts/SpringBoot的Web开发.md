---
title: SpringBoot的Web开发
date: 2019-03-29 18:54:45
tags: SpringBoot
categories: SpringBoot
---

## 简介

Spring Boot 提供了 spring-boot-starter-web 为 Web 开发予以支持，spring-boot-starter- web 为我们提供了嵌入的Tomcat以及Spring MVC的依赖。而Web相关的自动配置存储在 spring-boot-autoconfigure.jar 的org.springfiramework.boot.autoconfigure.web

在当前Spring毫无疑问已经成为java后台对象管理标准框架，除了通过IOC能够管理我们的自定义对象的生命周期之外还提供了众多功能繁复的可配置功能模块。但同时带来了复杂的配置项，这对初学者而言简直是一种灾难。于是SpringBoot应运而生，Springboot的出现大大简化了配置，主要表现在消除了web.xml和依赖注入配置的整合，处处遵循规约大于配置的思想，将初学者在繁杂的配置项中解放出来，专注于业务的实现，而不需要去关注太底层的实现。当然，也可以自己手动添加Web.xml，因为对于高端玩家而言，很多时候配置项还是很有必要的。

![ABFqZd.png](https://s2.ax1x.com/2019/03/29/ABFqZd.png)

从这些文件名可以看出：

```
ServerPropertiesAutoConfiguration 和 ServerProperties 自动配置内嵌 Servlet 容器； HttpEncodingAutoConfiguration 和 HttpEncodingProperties 用来自动配置 http 的编码； MultipartAutoConfiguration 和 MultipartProperties 用来自动配置上传文件的属性； JacksonHttpMessageConvertersConfiguration 用来自动配置 mappingJackson2Http MessageConverter 和 mappingJackson2XmlHttpMessage Converter; 
WebMvcAutoConfiguration 和 WebMvcProperties 配置 Spring MVC
```

## 工程结构

![ABkQeJ.png](https://s2.ax1x.com/2019/03/29/ABkQeJ.png)

```java
package com.hph.springboot.controller;


import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello() {
        return "Hello SpringBoot";
    }
}
```

![ABkweH.png](https://s2.ax1x.com/2019/03/29/ABkweH.png)

## 静态资源映射

在springboot中引入静态资源我们可是以用https://www.webjars.org/该网站上的配置,以jquery为例

在pom中添加这样我们就引入了jquery的静态资源

```xml
        <!--引入jquery-web-->
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.3.1</version>
        </dependency>
```

![ABkq6U.png](https://s2.ax1x.com/2019/03/29/ABkq6U.png)

这样就在Maven中添加了jquery的依赖那么我们该如何访问呢让我们探究以下一种的秘密:

观察WebMvcAutoConfiguration中的addResourceHandlers方法

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties implements ResourceLoaderAware {
  //可以设置和静态资源有关的参数，缓存时间等
```



```java
	@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Integer cachePeriod = this.resourceProperties.getCachePeriod();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry
						.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(cachePeriod));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
 			//静态资源文件夹映射
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler(staticPathPattern)
								.addResourceLocations(
										this.resourceProperties.getStaticLocations())
								.setCachePeriod(cachePeriod));
			}
		}
        //配置欢迎页映射
		@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(
				ResourceProperties resourceProperties) {
			return new WelcomePageHandlerMapping(resourceProperties.getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
		}

       //配置喜欢的图标
		@Configuration
		@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
		public static class FaviconConfiguration {

			private final ResourceProperties resourceProperties;

			public FaviconConfiguration(ResourceProperties resourceProperties) {
				this.resourceProperties = resourceProperties;
			}

			@Bean
			public SimpleUrlHandlerMapping faviconHandlerMapping() {
				SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
				mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
              	//所有  **/favicon.ico 
				mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
						faviconRequestHandler()));
				return mapping;
			}

			@Bean
			public ResourceHttpRequestHandler faviconRequestHandler() {
				ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
				requestHandler
						.setLocations(this.resourceProperties.getFaviconLocations());
				return requestHandler;
			}

		}

```

通过分析得知webjars以jar包的方式引入静态资源,因此我们可以访问

![ABAwcT.png](https://s2.ax1x.com/2019/03/29/ABAwcT.png)

 所有 /webjars/** ，都去 classpath:/META-INF/resources/webjars/ 找资源；

"/**" 访问当前项目的任何资源，都去（静态资源的文件夹）找映射

```
"classpath:/META-INF/resources/", 
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/" 
"/"：当前项目的根路径
```

localhost:8080/abc   去静态资源文件夹里面找abc

![ABE4Gq.png](https://s2.ax1x.com/2019/03/29/ABE4Gq.png)

**注意 :** 如果在添加静态资源之后访问静态资源返回ERROR可重启以下IDEA。

欢迎页； 静态资源文件夹下的所有index.html页面；被"/**"映射；

![ABVQeg.png](https://s2.ax1x.com/2019/03/29/ABVQeg.png)

所有的 **/favicon.ico  都是在静态资源文件下找；

改变图标 在resources目录下添加favicon.ico文件

![ABVXnS.png](https://s2.ax1x.com/2019/03/29/ABVXnS.png)





