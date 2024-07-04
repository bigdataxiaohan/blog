---
title: 微服务与SpringCloud
date: 2019-04-18 19:29:15
categories: 微服务
tags: 
 - 技术选型
---

## 微服务简介

“微服务架构”一词是在过去几年里涌现出来的，它用于描述一种独立部署的软件应用设计方式。这种架构方式并没有非常明确的定义，但有一些共同的特点就是围绕在业务能力、自动化布署、端到端的整合以及语言和数据的分散控制上面。目前为止,微服务我们也不太好给一个定义,但是绝大部分的微服务都有相似的特点。

一种架构⻛风格，将单体应⽤用划分成一组⼩的服务，服务之间相互协作，实现业务功能。 

 每个服务运⾏行在独⽴立的进程中，服务间采⽤用轻量量级的通信机制协作（通常是HTTP/ JSON） 

每个服务围绕业务能力力进⾏行行构建，并且能够通过⾃自动化机制独⽴立地部署 

很少有集中式的服务管理，每个服务可以使⽤用不不同的语⾔言开发，使⽤用不不同的存储技术 

 Loosely coupled service oriented architecture with bounded context ：基于有界上下⽂文的，松散耦合的⾯面向服务的架构  																	————Adrian Cockcroft。

 参考：https://www.martinfowler.com/articles/microservices.html

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/%E6%9E%B6%E6%9E%84/20190422161622.png)



## 微服务技术栈

| 微服务条目                             | 落地技术                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| 服务开发                               | Springboot、Spring、SpringMVC                                |
| 服务配置与管理                         | Netflix公司的Archaius、阿里的Diamond等                       |
| 服务注册与发现                         | Eureka、Consul、Zookeeper等                                  |
| 服务调用                               | Rest、RPC、gRPC                                              |
| 服务熔断器                             | Hystrix、Envoy等                                             |
| 负载均衡                               | Ribbon、Nginx等                                              |
| 服务接口调用(客户端调用服务的简化工具) | Feign等                                                      |
| 消息队列                               | Kafka、RabbitMQ、ActiveMQ等                                  |
| 服务配置中心管理                       | SpringCloudConfig、Chef等                                    |
| 服务路由（API网关）                    | Zuul等                                                       |
| 服务监控                               | Zabbix、Nagios、Metrics、Spectator等                         |
| 全链路追踪                             | Zipkin，Brave、Dapper等                                      |
| 服务部署                               | Docker、OpenStack、Kubernetes等                              |
| 数据流操作开发包                       | SpringCloud Stream（封装与Redis,Rabbit、Kafka等发送接收消息） |
| 事件消息总线                           | Spring Cloud Bus                                             |

## SpringCloud

![ESvKtf.png](http://hphblog.cn/SpringCloud/20190421180902.png)
SpringCloud，基于SpringBoot提供了一套微服务解决方案，包括服务注册与发现，配置中心，全链路监控，服务网关，负载均衡，熔断器等组件，除了基于NetFlix的开源组件做高度抽象封装之外，还有一些选型中立的开源组件。

SpringCloud利用SpringBoot的开发便利性巧妙地简化了分布式系统基础设施的开发，SpringCloud为开发人员提供了快速构建分布式系统的一些工具，包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等,它们都可以用SpringBoot的开发风格做到一键启动和部署。

SpringBoot并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过SpringBoot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包

SpringCloud=分布式微服务架构下的一站式解决方案，是各个微服务架构落地技术的集合体，俗称[微服务全家桶](<https://springcloud.cc/>)。

## 二者关系

SpringBoot专注于快速方便的开发单个个体微服务。

SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，
为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务。

SpringBoot可以离开SpringCloud独立使用开发项目，但是SpringCloud离不开SpringBoot，属于依赖的关系。

SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。



## 目前架构

分布式+服务治理Dubbo：应用服务化拆分+消息中间件。

![](<https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/%E6%9E%B6%E6%9E%84/20190422161756.png>)

## Dubbo和SpringCloud对比

![](<https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/%E6%9E%B6%E6%9E%84/20190422161809.png>)

**最大区别**：SpringCloud抛弃了Dubbo的RPC通信，采用的是基于HTTP的REST方式。严格来说，这两种方式各有优劣。虽然从一定程度上来说，后者牺牲了服务调用的性能，但也避免了上面提到的原生RPC带来的问题。而且REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这在强调快速演化的微服务环境下，显得更加合适。

**品牌机与组装机的区别**
很明显，Spring Cloud的功能比DUBBO更加强大，涵盖面更广，而且作为Spring的拳头项目，它也能够与Spring Framework、Spring Boot、Spring Data、Spring Batch等其他Spring项目完美融合，这些对于微服务而言是至关重要的。使用Dubbo构建的微服务架构就像组装电脑，各环节我们的选择自由度很高，但是最终结果很有可能因为一条内存质量不行就点不亮了，总是让人不怎么放心，但是如果你是一名高手，那这些都不是问题；而Spring Cloud就像品牌机，在Spring Source的整合下，做了大量的兼容性测试，保证了机器拥有更高的稳定性，但是如果要在使用非原装组件外的东西，就需要对其基础有足够的了解。

**社区支持与更新力度**：DUBBO停止了5年左右的更新，虽然2017.7重启了。对于技术发展的新需求，需要由开发者自行拓展升级（比如当当网弄出了DubboX），这对于很多想要采用微服务架构的中小软件组织，显然是不太合适的，中小公司没有这么强大的技术能力去修改Dubbo源码+周边的一整套解决方案，并不是每一个公司都有阿里的大牛+真实的线上生产环境测试过。

**定位**：Dubbo的定位始终是一款RPC框架，而Spring Cloud的目标是微服务框架下的一站式解决方案，在面临 微服务基础框架选型时Dubbo与Spring Cloud只能二选一。

选型依据： 整体解决方案和框架成熟度，社区热度，可维护性，学习曲线。

## SpringBoot和其他框架对比

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/架构/20190422223904.png)



## 微服务的利利和弊

| <font color="gree">利</font> | <font color="red">弊</font> |
| ---------------------------- | --------------------------- |
| 强模块化边界                 | 分布式系统复杂性            |
| 可独⽴立部署                 | 最终一致性                  |
| 技术多样性                   | 运维复杂性                  |
|                              | 测试复杂性                  |

## 康威法则

中文直译大概的意思就是：设计系统的组织，其产生的设计等同于组织之内、组织之间的沟通结构。看看下面的图片，再想想Apple的产品、微软的产品设计，就能形象生动的理解这句话。

微服务很多核心理念其实在半个世纪前的一篇文章中就被阐述过了，而且这篇文章中的很多论点在软件开发飞速发展的这半个世纪中竟然一再被验证，这就是[康威定律（Conway's Law）](http://www.melconway.com/Home/Conways_Law.html?spm=a2c4e.11153940.blogcont8611.5.7ea872f09xxfIQ)。

在康威的这篇文章中，最有名的一句话就是：

> Organizations which design systems are constrained to produce designs which are copies of the communication structures of these organizations. - Melvin Conway(1967)

用通俗的说法就是：组织形式等同系统设计。

这里的系统按原作者的意思并不局限于软件系统。据说这篇文章最初投的哈佛商业评论，结果程序员屌丝的文章不入商业人士的法眼，无情被拒，康威就投到了一个编程相关的杂志，所以被误解为是针对软件开发的。最初这篇文章显然不敢自称定律（law），只是描述了作者自己的发现和总结。后来，在Brooks Law著名的人月神话中，引用这个论点，并将其“吹捧”成了现在我们熟知“康威定律”。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/架构/20190422223944.png)

### 康威定律详细介绍

Mike从他的角度归纳这篇论文中的其他一些核心观点，如下：

- 第一定律
    - Communication dictates design
    - 组织沟通方式会通过系统设计表达出来
- 第二定律
    - There is never enough time to do something right, but there is always enough time to do it over
    - 时间再多一件事情也不可能做的完美，但总有时间做完一件事情
- 第三定律
    - There is a homomorphism from the linear graph of a system to the linear graph of its design organization
    - 线型系统和线型组织架构间有潜在的异质同态特性
- 第四定律
    - The structures of large systems tend to disintegrate during development, qualitatively more so than with small systems
    - 大的系统组织总是比小系统更倾向于分解

## 何时引入微服务

目前正在学习极客时间的微服务20讲，杨波老师的建议一开始并不是要直接使用微服务的，它需要前期的基础设施投入，复杂性很高，反而会影响我们业务的开展， 比较倾向于开始，单块应用的方式，当架构师对业务越来越清晰的时候，业务越来越大，架构师不断地拆分最后演化成微服务架构。









