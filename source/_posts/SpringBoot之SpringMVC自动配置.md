---
title: SpringBoot之SpringMVC自动配置
date: 2019-03-29 22:39:12
tags: SpringMVC
categories: SpringBoot 
---

## 自动配置

Spring Boot 自动配置好了SpringMVC

以下是SpringBoot对SpringMVC的默认配置:（WebMvcAutoConfiguration）

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.
    - 自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象（View），视图对象决定如何渲染（转发？重定向？））
    - ContentNegotiatingViewResolver：组合所有的视图解析器的；
    - 如何定制：我们可以自己给容器中添加一个视图解析器；自动的将其组合进来；
- Support for serving static resources, including support for WebJars (see below).静态资源文件夹路径,webjars
- Static `index.html` support. 静态首页访问
- Custom `Favicon` support (see below).  favicon.ico
- 自动注册了 of `Converter`, `GenericConverter`, `Formatter` beans.
    - Converter：转换器；  public String hello(User user)：类型转换使用Converter
    - `Formatter`  格式化器；  2019.3.30===Date；

```java
		@Bean
		@ConditionalOnProperty(prefix = "spring.mvc", name = "date-format") //在文件中配置日期格式化的规则
		public Formatter<Date> dateFormatter() {
			return new DateFormatter(this.mvcProperties.getDateFormat());//日期格式化组件
		}
```

**自己添加的格式化器转换器，我们只需要放在容器中即可** 



- Support for **HttpMessageConverters** (see below).
    - HttpMessageConverter：SpringMVC用来转换Http请求和响应的；User---Json；
    - HttpMessageConverters是从容器中确定；获取所有的HttpMessageConverter；
    - 自己给容器中添加HttpMessageConverter，只需要将自己的组件注册容器中（@Bean,@Component）

Automatic registration of `MessageCodesResolver` (see below).定义错误代码生成规则

Automatic use of a `ConfigurableWebBindingInitializer` bean (see below).

我们可以配置一个ConfigurableWebBindingInitializer来替换默认的；（添加到容器）

```
初始化WebDataBinder；
请求数据=====JavaBean；
```

**org.springframework.boot.autoconfigure.web：web的所有自动场景；**

 If you want to keep Spring Boot MVC features, and you just want to add additional [MVC configuration](https://docs.spring.io/spring/docs/4.3.14.RELEASE/spring-framework-reference/htmlsingle#mvc) (interceptors, formatters, view controllers etc.) you can add your own `@Configuration` class of type `WebMvcConfigurerAdapter`, but **without** `@EnableWebMvc`. If you wish to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter` or `ExceptionHandlerExceptionResolver` you can declare a `WebMvcRegistrationsAdapter` instance providing such components.

  If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`.

## 修改SprigBoot的默认配置

 在WebMvcAutoConfiguration中springboot如果想要给容器配置一个HiddenHttpMethodFilter

```java
	@Bean
	@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)   //先判断容器中是否由该组件
	public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
		return new OrderedHiddenHttpMethodFilter();       //如果没有 new一个新的组件添加到容器中
	}
```

### 修改模式

- SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（ViewResolver）将用户配置的和自己默认的组合起来；
- 在SpringBoot中会有非常多的xxxConfigurer帮助我们进行扩展配置
- 在SpringBoot中会有很多的xxxCustomizer帮助我们进行定制配置

## 扩展SpringMVC

在实际的开发过程中,仅仅依靠Springboot的自动配置是远远不够的。在有SpringMVC配置文件的时候，我们可能像下面以配置

```xml
    <mvc:view-controller path="/hello" view-name="success"/>
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/hello"/>
            <bean></bean>
        </mvc:interceptor>
    </mvc:interceptors>
```

在SpringBoot的开发中我们可以编写一个配置类（@Configuration），是WebMvcConfigurerAdapter类型；不能标注@EnableWebMvc;

```java
package com.hph.springboot.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class MyConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //super.addViewControllers(registry);
        //相当于浏览器发送/hello请求,也来到成功页面
        registry.addViewController("/hellomvc").setViewName("success");
    }
}
```

这样做即保留了所有的自动配置也能用我们扩展的配置

![ABDvb8.png](https://s2.ax1x.com/2019/03/30/ABDvb8.png)

### 原理探究

- WebMvcAutoConfiguration是SpringMVC的自动配置类

- 在做其他自动配置时会导入；@Import(**EnableWebMvcConfiguration**.class)
- 容器中所有的WebMvcConfigurer都会一起起作用；
- 我们的配置类也会被调用；
- 效果：SpringMVC的自动配置和我们的扩展配置都会起作用；

```java
	public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {

		private final WebMvcProperties mvcProperties;

		private final ListableBeanFactory beanFactory;

		private final WebMvcRegistrations mvcRegistrations;

		public EnableWebMvcConfiguration(
				ObjectProvider<WebMvcProperties> mvcPropertiesProvider,
				ObjectProvider<WebMvcRegistrations> mvcRegistrationsProvider,
				ListableBeanFactory beanFactory) {
			this.mvcProperties = mvcPropertiesProvider.getIfAvailable();
			this.mvcRegistrations = mvcRegistrationsProvider.getIfUnique();
			this.beanFactory = beanFactory;
		}
```

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

	private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

	//从容器中获取所有的WebMvcConfigurer
	@Autowired(required = false)
	public void setConfigurers(List<WebMvcConfigurer> configurers) {
		if (!CollectionUtils.isEmpty(configurers)) {
			this.configurers.addWebMvcConfigurers(configurers);           
		}
	}
```

### 参考实现

```java
//一个参考实现；将所有的WebMvcConfigurer相关配置都来一起调用；  
@Override
public void addViewControllers(ViewControllerRegistry registry) {
	for (WebMvcConfigurer delegate : this.delegates) {
  			delegate.addViewControllers(registry);
	 }
}
```

该方法时是将所有的WebMvcConfigurer相关配置都会一起调用。也就是我们自己写的配置类也会被调用。实现的效果就是：springmvc自身的配置起作用的时候把我们的自定义配置也遍历进来了SpringMVC的自动配置和我们的扩展配置都会起作用，就是我们所说的拓展

## 全面接管 SpringMVC

SpringBoot对SpringMVC的全部配置不在需要了，所有的配置都是我们自己来配置，只需要在配置类中添加一个@EnableWebMvc

```java
package com.hph.springboot.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@EnableWebMvc
@Configuration
public class MyConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //super.addViewControllers(registry);
        //相当于浏览器发送/hello请求,也来到成功页面
        registry.addViewController("/hellomvc").setViewName("success");
    }
}
```

![ABrn54.png](https://s2.ax1x.com/2019/03/30/ABrn54.png)

当添加@EnableWebMvc的时候SpringMVC的自动配置都失效了

### 建议

如果只是做一些简单的功能，不需要那么强大的功能我们可以使用全面接管，

### 原理

为什么@EnableWebMvc自动配置就失效了呢

进入@EnableWebMvc

```java
@EnableWebMvc
@Configuration
public class MyConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //super.addViewControllers(registry);
        //相当于浏览器发送/hello请求,也来到成功页面
        registry.addViewController("/hellomvc").setViewName("success");
    }
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

进入WebMvcAutoConfiguration的

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
		WebMvcConfigurerAdapter.class })
//容器中没有这个组件的时候接下来的自动的自动配置来才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		ValidationAutoConfiguration.class })
```

EnableWebMvc将WebMvcConfigurationSupport组件导入进来了，这个组件只是SpringMVC的基本功能；













