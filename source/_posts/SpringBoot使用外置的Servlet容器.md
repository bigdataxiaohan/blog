---
title: SpringBoot使用外置的Servlet容器
date: 2019-04-03 16:47:47
tags: JSP
categories: SpringBoot
---

嵌入式Servlet容器：应用打成可执行的jar

优点：简单、便携；

缺点：默认不支持JSP、优化定制比较复杂（使用定制器【ServerProperties、自定义EmbeddedServletContainerCustomizer】，自己编写嵌入式Servlet容器的创建工厂【EmbeddedServletContainerFactory】）；

![AgShnO.png](https://s2.ax1x.com/2019/04/03/AgShnO.png)

注意打包为War包

![AgSLgP.png](https://s2.ax1x.com/2019/04/03/AgSLgP.png)

外置的Servlet容器：外面安装Tomcat---应用war包的方式打包；

编写一个**SpringBootServletInitializer**的子类，并调用configure方法

```java
public class ServletInitializer extends SpringBootServletInitializer {

   @Override
   protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
       //传入SpringBoot应用的主程序
      return application.sources(SpringBoot04WebJspApplication.class);
   }
}
```

启动服务器就可以使用；

但是SpringBoot默认的嵌入容器Tomcat不支持JSP,因此我们需要用外置的Tomcat来打包该项目

![AgmXJf.png](https://s2.ax1x.com/2019/04/03/AgmXJf.png)

![Agnmy4.png](https://s2.ax1x.com/2019/04/03/Agnmy4.png)

![AgnMwR.png](https://s2.ax1x.com/2019/04/03/AgnMwR.png)

![Agnd0A.png](https://s2.ax1x.com/2019/04/03/Agnd0A.png)

## 原理

jar包：执行SpringBoot主类的main方法，启动ioc容器，创建嵌入式的Servlet容器；

war包：启动服务器，**服务器启动SpringBoot应用**【SpringBootServletInitializer】，启动ioc容器；

在servlet3.0中的第8.3 JSP container pluggability章节中

[文档链接](http://note.youdao.com/noteshare?id=9546758239c94cdc156b93f96b8aaf05&sub=1CC193D2303942AD8BDEAEC0A141DABA)

The ServletContainerInitializer class is looked up via the jar services API. For each application, an instance of the ServletContainerInitializer is created by the container at application startup time. The framework providing an 
implementation of the ServletContainerInitializer MUST bundle in the META-INF/services directory of the jar file a file called javax.servlet.ServletContainerInitializer, as per the jar services API, that points to the implementation class of the ServletContainerInitializer.In addition to the ServletContainerInitializer we also have an annotation -HandlesTypes. The HandlesTypes annotation on the implementation of the ServletContainerInitializer is used to express interest in classes that may have annotations (type, method or field level annotations) specified in the value of  the HandlesTypes or if it extends / implements one those classes anywhere in the class’ super types. The HandlesTypes annotation is applied irrespective of the setting of metadata-complete.

解读:

​	1）、服务器启动（web应用启动）会创建当前web应用里面每一个jar包里面ServletContainerInitializer实例：

​	2）、ServletContainerInitializer的实现放在jar包的META-INF/services文件夹下，有一个名为javax.servlet.ServletContainerInitializer的文件，内容就是ServletContainerInitializer的实现类的全类名

​	3）、还可以使用@HandlesTypes，在应用启动的时候加载我们感兴趣的类；

## 流程

1）、启动Tomcat

2）、org/springframework/spring-web/5.1.5.RELEASE/spring-web-5.1.5.RELEASE.jar!/META-INF/services/javax.servlet.ServletContainerInitializer：

Spring的web模块里面有这个文件：**org.springframework.web.SpringServletContainerInitializer**

3）、SpringServletContainerInitializer将@HandlesTypes(WebApplicationInitializer.class)标注的所有这个类型的类都传入到onStartup方法的Set<Class<?>>；为这些WebApplicationInitializer类型的类创建实例；

4）、每一个WebApplicationInitializer都调用自己的onStartup；

```java
public interface WebApplicationInitializer {
	void onStartup(ServletContext servletContext) throws ServletException;
}

```

![AgYWFg.png](https://s2.ax1x.com/2019/04/03/AgYWFg.png)

5）、相当于我们的SpringBootServletInitializer的类会被创建对象，并执行onStartup方法

6）、SpringBootServletInitializer实例执行onStartup的时候会createRootApplicationContext；创建容器

```java
protected WebApplicationContext createRootApplicationContext(
			ServletContext servletContext) {
        //1、创建SpringApplicationBuilder
		SpringApplicationBuilder builder = createSpringApplicationBuilder();
		builder.main(getClass());
		ApplicationContext parent = getExistingRootWebApplicationContext(servletContext);
		if (parent != null) {
			this.logger.info("Root context already created (using as parent).");
			servletContext.setAttribute(
					WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, null);
			builder.initializers(new ParentContextApplicationContextInitializer(parent));
		}
		builder.initializers(
				new ServletContextApplicationContextInitializer(servletContext));
		builder.contextClass(AnnotationConfigServletWebServerApplicationContext.class);		    	    //调用configure方法，子类重写了这个方法，将SpringBoot的主程序类传入了进来
		builder = configure(builder);
    
		builder.listeners(new WebEnvironmentPropertySourceInitializer(servletContext));
        //使用builder创建一个Spring应用
		SpringApplication application = builder.build();
		if (application.getAllSources().isEmpty() && AnnotationUtils
				.findAnnotation(getClass(), Configuration.class) != null) {
			application.addPrimarySources(Collections.singleton(getClass()));
		}
		Assert.state(!application.getAllSources().isEmpty(),
				"No SpringApplication sources have been defined. Either override the "
						+ "configure method or add an @Configuration annotation");
		// Ensure error pages are registered
		if (this.registerErrorPageFilter) {
			application.addPrimarySources(
					Collections.singleton(ErrorPageFilterConfiguration.class));
		}
        //启动Spring应用
		return run(application);
	}
```

Spring的应用就启动并且创建IOC容器

```java
	protected WebApplicationContext run(SpringApplication application) {
        //点击进入run方法
		return (WebApplicationContext) application.run();
	}

	private ApplicationContext getExistingRootWebApplicationContext(
			ServletContext servletContext) {
		Object context = servletContext.getAttribute(
				WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
		if (context instanceof ApplicationContext) {
			return (ApplicationContext) context;
		}
		return null;
	}
```

点击进入run方法

```java
	public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(
					SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
            
              //刷新IOC容器
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}

```







