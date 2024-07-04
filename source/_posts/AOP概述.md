---
title: AOP概述
date: 2019-03-03 16:18:34
tags: AOP
categories: JavaWeb
---

## AOP简介

AOP(Aspect-Oriented Programming，**面向切面编程**)：是一种新的方法论，是对传统 OOP(Object-Oriented Programming，面向对象编程)的补充。 AOP编程操作的主要对象是切面(aspect)，而切面**模块化横切关注点**。在应用AOP编程时，仍然需要定义公共功能，但可以明确的定义这个功能应用在哪里，以什么方式应用，并且不必修改受影响的类。这样一来横切关注点就被模块化到特殊的类里——这样的类我们通常称之为“切面”。 

AOP的好处： 每个事物逻辑位于一个位置，代码不分散，便于维护和升级业务模块更简洁，只包含核心业务代码

![kL6929.png](https://s2.ax1x.com/2019/03/03/kL6929.png)

## AOP术语

 横切关注点：从每个方法中抽取出来的同一类非核心业务。

切面(Aspect)：封装横切关注点信息的类，每个关注点体现为一个通知方法。

通知(Advice)：切面必须要完成的各个具体工作

目标(Target)：被通知的对象

代理(Proxy)：向目标对象应用通知之后创建的代理对象

连接点(Joinpoint)：横切关注点在程序代码中的具体体现，对应程序执行的某个特定位置。例如：类某个方法调用前、调用后、方法捕获到异常后等。

在应用程序中可以使用横纵两个坐标来定位一个具体的连接点：

![kL6u2d.png](https://s2.ax1x.com/2019/03/03/kL6u2d.png)

切入点：定位连接点的方式。每个类的方法中都包含多个连接点，所以连接点是类中客观存在的事物。如果把连接点看作数据库中的记录，那么切入点就是查询条件——AOP可以通过切入点定位到特定的连接点。切点通过org.springframework.aop.Pointcut 接口进行描述，它使用类和方法作为连接点的查询条件。

![kL6QKI.png](https://s2.ax1x.com/2019/03/03/kL6QKI.png)

##  AspectJ

AspectJ：Java社区里最完整最流行的AOP框架。在Spring2.0以上版本中，可以使用基于AspectJ注解或基于XML配置的AOP。

AspectJ是AOP的Java实现版本，定义了AOP的语法，可以说是对Java的一个扩展。相对于Java，AspectJ引入了join point(连接点)的概念，同时引入三个新的结构，pointcut(切点)， advice(通知)，inter-type declaration(跨类型声明)以及aspect。其中pointcut和advice是AspectJ中动态额部分，用来指定在什么条件下切断执行，以及采取什么动作来实现切面操作。这里的pointcut就是用来定义什么情况下进行横切，而advice则是指横切情况下我们需要做什么操作，pointcut和advice会动态的影响程序的运行流程。从某种角度上说，pointcut(切点)和我们平时用IDE调试程序时打的断点很类似，当程序执行到我们打的断点的地方的时候(运行到满足我们定义的pointcut的语句的时候，也就是join point连接点)，

&lt;aop:aspectj-autoproxy&gt; ：当Spring IOC容器侦测到bean配置文件中的&lt;aop:aspectj-autoproxy&gt;元素时，会自动为与AspectJ切面匹配的bean创建代理

用AspectJ注解声明切面

- 要在Spring中声明AspectJ切面，只需要在IOC容器中将切面声明为bean实例。
- 当在Spring IOC容器中初始化AspectJ切面之后，Spring IOC容器就会为那些与 AspectJ切面相匹配的bean创建代理。
-  在AspectJ注解中，切面只是一个带有@Aspect注解的Java类，它往往要包含很多通知。
- 通知是标注有某种注解的简单的Java方法。
- AspectJ支持5种类型的通知注解：

@Before：前置通知，在方法执行之前执行

 @After：后置通知，在方法执行之后执行

@AfterRunning：返回通知，在方法返回结果之后执行

@AfterThrowing：异常通知，在方法抛出异常之后执行

@Around：环绕通知，围绕着方法执行

## 切入点表达式

需要用到的包：链接：https://pan.baidu.com/s/1gmEusK9hXAaJvcB9_6XZPg  提取码：m3i9 

通过**表达式的方式**定位**一个或多个**具体的连接点。

语法细节:

```
execution([权限修饰符] [返回值类型] [简单类名/全类名] [方法名]([参数列表]))
```

### 准备

```java
package com.hph.spring.aspectJ.annotation;


public interface ArithmeticCalculator {
	
	public int add(int i ,int j );
	
	public int sub(int i, int j );
	
	public int mul(int i ,int j );
	
	public int div(int i, int j );
}

```

```java
package com.hph.spring.aspectJ.annotation;


import org.springframework.stereotype.Component;

@Component 	//托管给Spring
public class ArithmeticCalculatorImpl implements ArithmeticCalculator {

    @Override
    public int add(int i, int j) {
        int result = i + j;
        return result;
    }

    @Override
    public int sub(int i, int j) {
        int result = i - j;
        return result;
    }

    @Override
    public int mul(int i, int j) {
        int result = i * j;
        return result;
    }

    @Override
    public int div(int i, int j) {
        int result = i / j;
        return result;
    }

}	
```

### 通知

概述:  在具体的连接点上要执行的操作。 一个切面可以包括一个或者多个通知。通知所使用的注解的值往往是切入点表达式。

**前置通知 **：在方法执行之前执行的通知     使用@Before注解

```java
package com.hph.spring.aspectJ.annotation;


import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

import java.util.Arrays;

/**
 * 日志切面
 */
@Component
@Aspect
public class LoggingAspect {
    // 前置通知:再目标方法(连接点)执行之前执行
    @Before("execution(public  int com.hph.spring.aspectJ.annotation.ArithmeticCalculatorImpl.add(int ,int ))")
    public void beforeMethod(JoinPoint joinPoint) {
        //获取方法的参数
        Object[] args = joinPoint.getArgs();
        //方法的名字
        String metodName = joinPoint.getSignature().getName();
        System.out.println("LoggingAspct ==> The method" + metodName + "   begin with " + Arrays.asList(args));
    }

}

```

```java
package com.hph.spring.aspectJ.annotation;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {

        ApplicationContext ctx = new ClassPathXmlApplicationContext("spring-aspectJ_annotation.xml");
        ArithmeticCalculator ac = ctx.getBean("arithmeticCalculatorImpl", ArithmeticCalculator.class);
        int result = ac.add(1, 1);
        System.out.println(result + "\n代理对象: " + ac.getClass().getName());
    }
}
```



**后置通知 **：后置通知：后置通知是在连接点完成之后执行的，即连接点返回结果或者抛出异常的时候   使用@After注解

```java

    /**
     * 后置通知:目标方法之后执行:不管目标方法是否抛出异常都会执行 不会获取到方法的结果
     */
    //  *: 任意修饰符 任意返回值类型           * 任意方法 ..:任意参数列表
    @After("execution(* com.hph.spring.aspectJ.annotation.ArithmeticCalculatorImpl.*(..))")
    //连接点对象
    public void afterMethod(JoinPoint joinPoint) {
        //方法名字
        String methodName = joinPoint.getSignature().getName();

        System.out.println("LoggingAspct ==> method " + methodName + "ends");
    }
```

**返回通知  **：无论连接点是正常返回还是抛出异常，后置通知都会执行。如果只想在连接点返回的时候记录日志，应使用返回通知代替后置通知。使用@AfterReturning注解,在返回通知中访问连接点的返回值

```java
/**
     * 返回通知:再目标方法正常执行结束后执行 可以获取方法的返回值
     * returing 的值 一定要和 形参 的值一致
     */
    @AfterReturning(value = "execution(* com.hph.spring.aspectJ.annotation.ArithmeticCalculatorImpl.*(..))", returning = "result")
    public void afterReturningMethod(JoinPoint joinPoint, Object result) {
        //方法名字
        String methodname = joinPoint.getSignature().getName();
        System.out.println("返回通知 LoggingAspct ==> The method " + methodname + "end with:" + result);
    }
```

在返回通知中，只要将returning属性添加到@AfterReturning注解中，就可以访问连接点的返回值。该属性的值即为用来传入返回值的参数名称

必须在通知方法的签名中添加一个同名参数。在运行时Spring AOP会通过这个参数传递返回值原始的切点表达式需要出现在pointcut属性中

**异常通知**：只在连接点抛出异常时才执行异常通知将throwing属性添加到@AfterThrowing注解中，也可以访问连接点抛出的异常。Throwable是所有错误和异常类的顶级父类，所以在异常通知方法可以捕获到任何错误和异常。

```java
    /**
     * 异常通知:再目标方法抛出异常后执行
     * 获取方法的异常: 通过throwing 来指定一个名字,必须要与一个参数名一致.
     * 可以通过形参中异常的类型才会执行异常通知
     */
    @AfterThrowing(value = "execution(* com.hph.spring.aspectJ.annotation.ArithmeticCalculatorImpl.*(..))", throwing = "ex")
    //如果指定的异常类型与抛出异常类型不匹配则会抛出
    public void afterThrowingMethod(JoinPoint joinPoint, ArithmeticException ex) {
        //获取方法名称
        String methodname = joinPoint.getSignature().getName();
        System.out.println("异常通知: LoggingAspect ==>t Thew method" + methodname + "occurs Exception " + ex);
    }
```

如果只对某种特殊的异常类型感兴趣，可以将参数声明为其他异常的参数类型。然后通知就只在抛出这个类型及其子类的异常时才被执行

**环绕通知**：环绕通知是所有通知类型中功能最为强大的，能够全面地控制连接点，甚至可以控制是否执行连接点。 

```java
  /**
     * 环绕通知:环绕着目标方法执行 可以理解为前置 后置 返回 异常 通知的结合体
     */
    //对于环绕通知必须声明一个Object的目标方法执行
    @Around("execution(* com.hph.spring.aspectJ.annotation.ArithmeticCalculatorImpl.*.*(..))")
    public Object arroundMethod(ProceedingJoinPoint pjp) {

        //执行目标方法
        try {
            //前置通知
            Object result = pjp.proceed();
            //返回通知
            return result;
        } catch (Throwable e) {
            //异常通知
            e.printStackTrace();
        } finally {
            //后置通知
        }
        return null;
    }
}
```



对于环绕通知来说，连接点的参数类型必须是ProceedingJoinPoint。它是 JoinPoint的子接口，允许控制何时执行，是否执行连接点。

在环绕通知中需要明确调用ProceedingJoinPoint的proceed()方法来执行被代理的方法。如果忘记这样做就会导致通知被执行了，但目标方法没有被执行。

```java
@Around("execution(* com.hph.spring.aspectJ.annotation.ArithmeticCalculatorImpl.*(..))")
    public Object arroundMethod(ProceedingJoinPoint pjp) {
        //执行目标方法
        try {
            //前置通知
            Object[] args = pjp.getArgs();
            String metodName = pjp.getSignature().getName();
            System.out.println("Arround前置通知: LoggingAspct ==> The method" + metodName + "   begin with " + Arrays.asList(args));
            Object result = pjp.proceed();
            //返回通知
            String methodname = pjp.getSignature().getName();
            System.out.println("Arround返回通知 LoggingAspct ==> The method " + methodname + "end with:" + result);
        } catch (Throwable e) {
            //异常通知
            e.printStackTrace();
            String methodname = pjp.getSignature().getName();
            System.out.println("Arround异常通知: LoggingAspect ==>t Thew method" + methodname + "occurs Exception " + e);
        } finally {
            //后置
            String methodName = pjp.getSignature().getName();
            System.out.println("Arround后置通知: LoggingAspct ==> method " + methodName + "ends");
        }
        return null;
    }
}
```



**注意：环绕通知的方法需要返回目标方法执行之后的结果，即调用 joinPoint.proceed();的返回值，否则会出现空指针异常。**

### 重用切入点

在编写AspectJ切面时，可以直接在通知注解中书写切入点表达式。但同一个切点表达式可能会在多个通知中重复出现。

在AspectJ切面中，可以通过@Pointcut注解将一个切入点声明成简单的方法。切入点的方法体通常是空的，因为将切入点定义与应用程序逻辑混在一起是不合理的。

切入点方法的访问控制符同时也控制着这个切入点的可见性。如果切入点要在多个切面中共用，最好将它们集中在一个公共的类中。在这种情况下，它们必须被声明为public。在引入这个切入点时，必须将类名也包括在内。如果类没有与这个切面放在同一个包中，还必须包含包名。

其他通知可以通过方法名称引入该切入点

```java
@Pointcut("execution(* *.*(..))")
private void loggingOperation() {
}

@Before("loggingOperation()")
public void logBefore(JoinPoint joinPoint)  {
    //业务逻辑
    Object[] args = joinPoint.getArgs();
    //方法的名字
    String metodName = joinPoint.getSignature().getName();
    System.out.println("重用切入点前置通知: LoggingAspct ==> The method" + metodName + "   begin with " + Arrays.asList(args));
}

@AfterReturning(pointcut = "loggingOperation()",returning = "result")
public void logAfterReturing(JoinPoint joinPoint,Object result ) {
    String methodname = joinPoint.getSignature().getName();
    System.out.println("重用切入点返回通知 LoggingAspct ==> The method " + methodname + "end with:" + result);
}
@AfterThrowing(pointcut = "loggingOperation()",throwing = "e")
public void logAfterThrowing(JoinPoint joinPoint,ArithmeticException e) {
    //获取方法名称
    String methodname = joinPoint.getSignature().getName();
    System.out.println("重用切入点返回通知:异常通知: LoggingAspect ==>t Thew method" + methodname + "occurs Exception " + e);
}
```

###  指定切面的优先级

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.util.Arrays;

@Component
@Aspect
@Order(0) //设置切面的优先级  0-2147483647 数字越小优先级越高
public class ValidationAspect {
    @Before("execution(* com.hph.spring.aspectJ.annotation.*.*(..))")
    public void beforeMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        System.out.println("优先级问题ValidationAspcet  ===>The method" + methodName + "begin with " + Arrays.asList(args));
    }
}
```



### XML配置

切入点使用&lt;aop:pointcut&gt;元素声明。

切入点必须定义在&lt;aop:aspect&gt;元素下，或者直接定义在<aop:config&gt;元素下。 ① 定义在&lt;aop:aspect&gt;元素下：只对当前切面有效② 定义在&lt;aop:config&gt;元素下：对所有切面都有效

基于XML的AOP配置不允许在切入点表达式中用名称引用其他切入点。

```xml
    <!--目标对象-->
    <bean id="arithmeticCalculatorImpl" class="com.hph.spring.aspectJ.xml.ArithmeticCalculatorImpl"></bean>

    <!--切面-->
    <bean id="loggingAspect" class="com.hph.spring.aspectJ.xml.LoggingAspect"></bean>
    <bean id="validationAspect" class="com.hph.spring.aspectJ.xml.ValidationAspect"></bean>
    <!--AOP:切面  通知 切入点表达式-->
    <aop:config>
        <!--切面-->
        <aop:aspect ref="loggingAspect">
            <!--切入点表达式-->
            <aop:pointcut id="myPointCut" expression="execution(* com.hph.spring.aspectJ.xml.*.*(..))"></aop:pointcut>
            <!--通知-->
            <aop:before method="beforeMethod" pointcut-ref="myPointCut"></aop:before>
            <aop:after method="afterMethod" pointcut-ref="myPointCut"></aop:after>
            <aop:after-returning method="afterReturningMethod" pointcut-ref="myPointCut" returning="result"></aop:after-returning>
            <aop:after-throwing method="afterThrowingMethod" pointcut-ref="myPointCut" throwing="ex"></aop:after-throwing>
            <aop:around method="arroundMethod" pointcut-ref="myPointCut"></aop:around>
        </aop:aspect>

    </aop:config>
```

在aop名称空间中，每种通知类型都对应一个特定的XML元素。

通知元素需要使用&lt;pointcut-ref&gt;来引用切入点，或用&lt;pointcut&gt;直接嵌入切入点表达式。

method属性指定切面类中通知方法的名称











