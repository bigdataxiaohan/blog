---
title: 消息队列RabbitMQ
date: 2019-04-11 10:13:39
tags: RabbitMQ
categories: SpringBoot
---

## 消息队列（Message Queue）

消息: 网络中的两台计算机或者两个通讯设备之间传递的数据。例如说：文本、音乐、视频等内容。

队列：一种特殊的线性表（数据元素首尾相接），特殊之处在于只允许在首部删除元素和在尾部追加元素。入队、出队。

消息队列：顾名思义，消息+队列，保存消息的队列。消息的传输过程中的容器；主要提供生产、消费接口供外部调用做数据的存储和获取。

## 消息队列分类

MQ分类：点对点（P2P）、发布订阅（Pub/Sub）

共同点：消息生产者生产消息发送到queue中，然后消息消费者从queue中读取并且消费消息。

不同点：    P2P模型包含：消息队列(Queue)、发送者(Sender)、接收者(Receiver)一个生产者生产的消息只有一个消费者(Consumer)（即一旦被消费，消息就不在消息队列中）。打电话。

Pub/Sub包含：消息队列(Queue)、主题（Topic）、发布者（Publisher）、订阅者（Subscriber）

每个消息可以有多个消费者，彼此互不影响。比如我发布一个微博：关注我的人都能够看到。

## 消息队列模式

1. 点对点模式（一对一，消费者主动拉取数据，消息收到后消息清除）

点对点模型通常是一个基于拉取或者轮询的消息传送模型，这种模型从队列中请求信息，而不是将消息推送到客户端。这个模型的特点是发送到队列的消息被一个且只有一个接收者接收处理，即使有多个消息监听者也是如此。

2. 发布/订阅模式（一对多，数据生产后，推送给所有订阅者）

发布订阅模型则是一个基于推送的消息传送模型。发布订阅模型可以有多种不同的订阅者，临时订阅者只在主动监听主题时才接收消息，而持久订阅者则监听主题的所有消息，即使当前订阅者不可用，处于离线状态。

## 消息队列的好处

- 解耦：允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

![A7YSKK.png](https://s2.ax1x.com/2019/04/11/A7YSKK.png)

- 冗余：消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。
- 扩展性：因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。
- 灵活性 & 峰值处理能力： 在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

![A7YAPA.png](https://s2.ax1x.com/2019/04/11/A7YAPA.png)

- 可恢复性：系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。
- 顺序保证：在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。（Kafka保证一个Partition内的消息的有序性）
- 缓冲：有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。
- 异步通信：很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

![A7Y9bD.png](https://s2.ax1x.com/2019/04/11/A7Y9bD.png)



JMS（Java Message Service）JAVA消息服务：基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

AMQP（Advanced Message Queuing Protocol）高级消息队列协议，也是一个消息代理的规范，兼容JMS RabbitMQ是AMQP的实现

![A7YHQP.png](https://s2.ax1x.com/2019/04/11/A7YHQP.png)

消息队列还有Kafaka也可以在本站中搜索阅读。

### RabbitMQ

### 简介

RabbitMQ是实现了高级消息队列协议（AMQP）的开源消息代理软件（亦称面向消息的中间件）。RabbitMQ服务器是用Erlang语言编写的，而集群和故障转移是构建在开放电信平台框架上的。所有主要的编程语言均有与代理接口通讯的客户端库。

###  核心概念

#### Message

消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

![A7tW60.png](https://s2.ax1x.com/2019/04/11/A7tW60.png)

#### Publisher

消息的生产者，也是一个向交换器发布消息的客户端应用程序。

#### Exchange

交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

Exchange有4种类型：direct(默认)，fanout, topic, 和headers，不同类型的Exchange转发消息的策略有所区别

#### Queue

消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

#### Binding

绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

Exchange 和Queue的绑定可以是多对多的关系。

#### Connection

网络连接，比如一个TCP连接。

#### Channel

信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁
TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条TCP 连接。

#### Consumer

消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

#### Virtual Host

虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个
vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

####  Broker

表示消息队列服务器实体

### 运行机制

AMQP 中的消息路由:

AMQP中消息的路由过程和Java开发者熟悉的JMS存在一些差别，AMQP中增加了**Exchange**和**Binding**的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。

![A7tvnK.png](https://s2.ax1x.com/2019/04/11/A7tvnK.png)

#### Exchange

**Exchange**分发消息时根据类型的不同分发策略有区别，目前共四种类型：**direct**、**fanout**、**topic**、**headers** 。headers 匹配 AMQP 消息的 header 而不是路由键， headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：

##### Direct

![A7NyDK.png](https://s2.ax1x.com/2019/04/11/A7NyDK.png)

消息中的路由键（routingkey）如果和 Binding中的 bindingkey 一致，交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。它是完全匹配、单播的模式。

##### Fanout

![A7Nh8A.png](https://s2.ax1x.com/2019/04/11/A7Nh8A.png)

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout类型转发消息是最快的。

##### Topic

![A7NL5Q.png](https://s2.ax1x.com/2019/04/11/A7NL5Q.png)
交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些**单词之间用点隔开**。它同样也会识别两个通配符：符号`#`和符号`*`。`#`匹配**0**个或多个单词，`*`匹配一个单词。

## 准备

### Docker安装rabbitmq

![A7a6te.png](https://s2.ax1x.com/2019/04/11/A7a6te.png)

![A7dJDP.png](https://s2.ax1x.com/2019/04/11/A7dJDP.png)

运行成功

### 登录

默认的账号密码都为guest

![A7dDvn.png](https://s2.ax1x.com/2019/04/11/A7dDvn.png)

![A7dgET.png](https://s2.ax1x.com/2019/04/11/A7dgET.png)

![A708Tf.png](https://s2.ax1x.com/2019/04/11/A708Tf.png)

![A704n1.png](https://s2.ax1x.com/2019/04/11/A704n1.png)

![A7BwCD.png](https://s2.ax1x.com/2019/04/11/A7BwCD.png)

![A7DVRe.png](https://s2.ax1x.com/2019/04/11/A7DVRe.png)

![A7DuqI.png](https://s2.ax1x.com/2019/04/11/A7DuqI.png)

![A7D0oV.png](https://s2.ax1x.com/2019/04/11/A7D0oV.png)

### 绑定

![A7DHQH.png](https://s2.ax1x.com/2019/04/11/A7DHQH.png)

![A7DOeI.png](https://s2.ax1x.com/2019/04/11/A7DOeI.png)

![A7sAjH.png](https://s2.ax1x.com/2019/04/11/A7sAjH.png)

### Direct

![A7y9qs.png](https://s2.ax1x.com/2019/04/11/A7y9qs.png)

只有一条匹配到了

![A7yEGT.png](https://s2.ax1x.com/2019/04/11/A7yEGT.png)

点对点模式只有一条消息我们来获取一下。

![A7yJzD.png](https://s2.ax1x.com/2019/04/11/A7yJzD.png)

### Fanount

![A7ywdI.png](https://s2.ax1x.com/2019/04/11/A7ywdI.png)

![A7yrJf.png](https://s2.ax1x.com/2019/04/11/A7yrJf.png)

所有队列都收到消息了

![A7ygyQ.png](https://s2.ax1x.com/2019/04/11/A7ygyQ.png)

### Topic

根据路由键的规则发送

![A7cl8K.png](https://s2.ax1x.com/2019/04/11/A7cl8K.png)

由于hphblog.news只与hph.news 和hphblog.news匹配因此我们可以在hph.new和hphblog.news中匹配到我们所想要匹配的消息。

![A7cGKe.png](https://s2.ax1x.com/2019/04/11/A7cGKe.png)

我们在换成其他的

![A7cgVs.png](https://s2.ax1x.com/2019/04/11/A7cgVs.png)

此时每个队列中的消息增加一条

![A7cR5q.png](https://s2.ax1x.com/2019/04/11/A7cR5q.png)



























