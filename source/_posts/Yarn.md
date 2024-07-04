---
title: Yarn
date: 2018-12-19 11:19:47
tags: Yarn
categories: 大数据
---

## YARN的介绍

Apache Hadoop YARN （Yet Another Resource Negotiator，另一种资源协调者）是一种新的 Hadoop 资源管理器，它是一个通用资源管理系统，可为上层应用提供统一的资源管理和调度，它的引入为集群在利用率、资源统一管理和数据共享等方面带来了巨大好处。

YARN是再MRv1发展过来的，它克服了NRv1的各种限制，因此我们先了解一下MRv1

### MRv1

![FBb5dA.md.png](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/FBb5dA.md.png)

JobTracker：用户程序提交了一个Job，任务（job）会发给JobTracker，JobTracker是Map-Reduce框架中心，它负责把任务分解成map和reduce的作业（task）；需要与集群中的机器定时心跳(heartbeat)通信；需要管理那些程序应该跑在那些机器上；需要管理所有job失败、重启操作。

Tasktracker是JobTracker和Task之间的桥梁，Tasktracker可以配置map和reduce的作业操(task slot)。TaskTracker通过心跳告知JobTracker自己还有空闲的作业Slot时，JobTrackr会向其分派任务。它将接收并执行JobTracker的各种命令:启动任务、提交任务、杀死任务。

TaskScheduler工作再JobTracker上。根据JobTracker获得的slot信息完成具体的分配工作，TaskScheduler支持多种策略以提高集群工作效率。

#### 局限性

1. 扩展局限性：JobTracker同时要兼顾资源管理和作业控制的两个功能，成为系统的一个瓶颈，制约Hadoop集群计算能力的拓展。（4000节点任务数400000）
2. MRv1采用master/slave结构，其中JobTracker作为计算管理的master存在单点问题，它容易出现故障整个MRv1不可用。
3. 资源利用率局限。MRv1采用了基于作业槽位（slot）的资源分配模型，槽位时一个粗粒度的资源分配单位，通常一个作业不会用完槽位对应的物理资源，使得其他作业也无法再利用空闲资源。对于一些IO密集，CPU空闲,当作业槽占满之后其他CPU密集型的也无法使用集群。MRv1将槽分为Map和Reduce两种slot，且不允许他们之间共享，这样做会导致另一种槽位资源紧张，另外一种资源闲置。
4. 无法支持多种计算框架，随着数据处理要求越来越精细，Mapreduce这种基于磁盘的离线计算框架已经满足不了各方需求，从而出现了新的计算框架，如内存计算框架、流式计算框架、迭代计算资源框架、而MRv1不支持多种计算框架并存。

### YARN

随着互联网的高速发展，新的计算框架不断出现，如内存计算框架、流式计算框架、迭代计算资源框架、这几种框架通常都会被用到考虑到资源的利用率运维和数据共享等因素，企业通常希望将所有的计算框架部署到一个 公共集群中，让他们共享集群的计算资源，并对资源进行同意使用，同时又能采用简单的资源隔离方案，这样便催生了轻量弹性计算平台需求Yarn的设计是一个弹性计算平台，不仅仅支持Mapreduce计算框架而是朝着对多种计算框架进行统一管理方向发展。

![FBvbzq.md.png](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/FBvbzq.md.png)

#### 优点

1. 资源利利用率高，按照框架角度进行资源划分，往往存在应用程序数据和计算资源需求的不均衡性，使得某段时间内计算资源紧张，而另外一种计算方式的资源空闲，共享集群模式则通过框架共享全部的计算资源，使得集群中的资源更加充分合理的利用。

2. 运维成本低，如果使用：”一个框架一个集群“的模式，运维人员需要独立管理多个框架，进而增加运维的难度，共享模式通常只需要少数管理员可以完成多个框架的管理。

3. 数据共享，随着数据量的增加，跨集群之间的数据不仅增加了硬件成本，而且耗费时间，共享集群模式可以共享框架和硬件资源，大大降低了数据移动带来的成本。

#### 组件

YARN采用了Master/Slave结构，在整个资源管理框架中ResourceManager为master，NodeManager为Slave。ResourceManager负责对各个 NodeManager上的资源进行统一的管理和调度，当用户提交一个应用程序时，需要生成以一个用于追踪和管理这个程序即ApplicationMaster，它负责向ResourceManager申请资源，并要求NodeManager启动可以占用一定的资源任务，不同的ApplicationMaster会被分不到不同的节点上，他们之间是相互独立的。

[![FDCkZV.md.png](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/FDCkZV.md.png)](https://imgchr.com/i/FDCkZV)

##### ResourceManager(RM)

ResourceManager负责整个系统的资源分配和管理，是一个全局的资源管理器。主要由两个组件构成：调度器和应用管理器。

1. 调度器(Scheduler)：调度器根据队列、容器等限制条件（每个队列配给  一定的资源，最多执行一定数量的作业等），将系统中的资源分配给各自正在运行的程序。调度器仅根据各个应用程序的资源需求进行资源分配，而资源分配单位用一个抽象的概念”资源容器“(Container)从而限定每个使用的资源量，容器用于执行的特定应用的进程每个容器都有所有资源限制（CPU 内存 等）一个容器可以是一个Unix进程也可以是一个Linux cgroup(Linux内核提升的一种限值，记录隔离进程并且使用的物理资源 CPU 内存 IO 等 谷歌提出)调度器相应应用程序的资源请求，是可插拔的，如Capcity  Scheduler、Fair Scheduler。
2. 应用程序管理器（Applications Manager）：应用程序管理器负责管理整个系统的所有应用程序，包括应用程序提交，与调度器协商资源以启动ApplicationMaster、监控ApplicationMaster运行状态并在失败时重新启动等，追踪分给的Container的情况。

##### ApplicationMaster(AM)

ApplicationMaster是一个详细的框架库，它结合了ResourceManager获取资源和NodeManager协同工作来运行和监控任务，用户提交的每一个应用程序军包含一个AM，主要的功能是：

1. 与ResourceManager调度器协商以获取抽象资源(Container);
2. 负责应用的监控，追踪应用执行状态重启任务失败等。
3. 与NodeManager协同工作完成Task的执行和监控。

MRappMaster是Mapreduce的ApplicationMaster实现，它是的Mapreduce应用程序可以直接在YARN平台之上，MRApplication负责Mappreduce应用的生命周期、作业管理资源申请再分配、container启动与释放、作业恢复等。其他的如计算框架如 Spark on YARN ,Storm on Yarn。如果需要可以自己写一个符合规范的YARN的应用模型。

##### NodeManager(NM)

NM是每个节点上的资源和任务管理器，他会定时地向RM汇报 本节点上地资源使用情况和各个Container地运行状态；同时会接受AM的Container启动/停止等请求。

##### Container

YARN 资源抽象 ，封存了某个节点是的多维度资源，如内存、CPU、磁盘、网络等。当AM向RM申请资源时，RM为AM返回资源是用Container表示的 。YARN为每一个任务分配给一个Container且该任务只能读取该Container中描述的资源。Container不同于MRv1的slot，他是一个动态划分单位，根据应用程序的需求动态生成的。

##### Yarn的工作流程

![FDkeTx.md.png](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/FDkeTx.md.png)

1. 用户向YARN提交应用程序，其中还包含了ApplicationMaster程序，启动AM的命令，命令参数和用户程序等等；本步骤需要准确描述运行AplicationMaster的unixs进程的所有信息。通常由YarnClient完成。
2. Resource Manager为该应用程序分配一个Container，和对应的NameNode进行通信，要求他在Container中启动ApplicationMaster；
3. ApplicationMaster首先向ResourceManager注册，这样用户可以直接通过RM查看程序的运行状态。然后它将为各个任务申请资源，并监控他的运行状态，知道运行结束，重复4~7；
4. ApplicationMaster采用轮询的方式通过RPC方式向ResourceManager申请和领取资源，资源的协调通过异步方式完成。
5. ApplicationMaster一旦申请资源后，便开始与对应的NodeManager通信，要求他们启动作业。
6. NodeManager为作业设置好运行环境（环境变量，jar包，二进制程序）将任务写道一个脚本中，并通过运行该脚本启动作业。
7. 各个作业通过协议RPC方式向ApplicationMaster同步汇报自己当前的进度和状态，AM随时掌握各个作业的运行状态，从而在作业失败的时候重启作业。
8. 应用程序运行完成后，ApplicationMaster向ResourceManager注销并关闭。

##### YARN资源模型

一个应用程序可以通过ApplicationMaster请求特定的资源需求来满足它的资源需要。调度器会被分配给一个Container来相应资源需求、用于满足ApplicationMaster在ResourceRequst中提出的要求。Container包含5类信息：优先级、期望资源所在节点、资源量、container数目、松弛本地性(是否没有满足本地资源时，选择机架本地资源)

ResourceRequst包含物种信息:

- 资源名称：期望所在的主机名、机架、用*表示没有特殊要求。
- 优先级： 程序内部请求的优先级，用来调整程序内部各个ResourceRequst的次序。
- 资源需求：需要的资源量，表示成内存里，CPU核数的元组(目前YARN仅支持内存和CPU两种资源)
- container数：表示 需要container数量，它决定了用该ResourceRequst指定的Container总数。
- 是否松弛本地性：在本机架资源剩余量无法满足要求是改为只要运行在服务器所在的机架上进行就行。

本质上Container是一种资源分配的形式，是ResourceManager为ResourceRequst成功分配资源的结果。Container授予节点在机器上使用资源（如内存、CPU）的权利，YARN允许的不仅仅是Java的应用程序，概念上Container在YARN分配任务分为两层：第一层为ResourceManage调度器结果分配给用用程序ApplicationMaster，第二层，作为运行环境用ApplicationMaster分配给作业资源。

![FDJTAK.md.png](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/FDJTAK.md.png)

​    									YARN启动应用程序流程

首先，客户端通知ResourceManager它要提交一个应用程序。随后ResourceManager在应答中返回一个ApplicationId以及必要的客户端请求资源的集群容量信息，资源请求分为两步：1.提交Application请求 2.返回ApplicationID。

3.客户端使用“Application submission Contex”发起响应，Application subminssion上下文包含了ApplicationID、客户名、队列以及其他启动Application所需要的信息。“ContainerLaunch Context”(CLC)也会发送给ResourceManager,CLC提供资源需求、作业文件、安全令牌以及在节点上启动ApplicationMaster所需要的其他信息。应用程序被提交之后，客户端可以向ResourceManager请求结束这个应用或者提供这个应用程序的状态。

4.当ResourceManager接收到客户端的应用提交上下文，它会为ApplcationMaster调度一个可用的Container来启动AM。如果合适的Container，Resource会联系相应的NodeManager并启动ApplicationMaster。如果没有合适的Container，则请求必须等待。ApplicationMaster会发送注册请求到ResourceManager，RM可以通过RPC的方式监控应用程序的状态。在注册请求中，ResourceManager会发送集群的最大和最小的容器容量(第5步)，供ApplicationMaster决定如何使用当前集群的可用资源。

基于ResourceMangaer的可用资源信息，ApplicationMaster会请求一定数量的Container（第6步），ResourceManager根据调度政策，尽量为Application分配资源，最为资源请求应答发送给ApplicationMaster（第7步）。

应用启动之后，ApplicationMaster将心跳和进度信息通过心跳发送给ResourceManager，这些心跳中，ApplicationMaster可以请求或释放Container。应用结束时ApplicationMaster发送给结束信息到ResourceManager以注销自己。

![FDU9N6.md.png](https://s1.ax1x.com/2018/12/19/FDU9N6.md.png)

ResourceManager已经将分配的NodeManager的控制权移交给ApplicationMaster。ApplicationMaster将独立联系其指定的节点管理器，并提供Container Launch Context，CLC包括环境便利，远程存储依赖文件、以及环境依赖文件，数据文件都被拷贝到节点的本地存储上。同一个节点的依赖文件都可以被同一应用程序Container共享。

一旦所有的Container都启动完成，ApplicationMaster就可以检查他们的状态。ResourceManager不参与应用程序的执行，只处理调度和监控其他资源。ResourceManager可以命令NodeManager杀死Container，比如ApplicationMaster通知ResourceManager自己的任务结束或者时ResourceManager需要为其他应用程序抢占资源。或者Container超出资源限制时都可能发生杀死Container事件。当Container被销毁后，NodeManager会清理它的本地工作目录。	应停用程序结束后，ApplicationMaster会发送结束信号通知ResourceManager，然后ResourceManager通知NodeManager收集日志并清理Container专用文件。如果Container还没退出，NodeManager也可以接收指令去杀死剩余的Container。

#### YARN 中的调度

#### FIFO

![FDdr7R.png](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/FDdr7R.png)

简单易懂，不需要任何配置(FIFO Scheduler),容量调度器(Capcity Scheduler)和公平调度器（Fair Scheduler）。FIFO调度器将应用放置在一个队列中然后按照提交的顺序(先进先出)运行应用，首先为队列中的第一个任务请求分配资源。第一个应用的请求被满足后再一次为队列的下一条进行服务。

#### Capacity

![FDdDB9.png](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/FDdDB9.png)

一个独立的专门队列保证小作业提交就可以启动，由于队列容量是为了那个队列中的作业所保留的。这意味这与使用FIFO调度器相比，大作业执行的事件要长。容量调度器允许多个组织共享一个Hadoop集群，每个组织可以分配到全部集群资源的一部分。每个组织都被配置一个专门的队列，每个队列都被配置为可以使用一定的集群资源。队列可以进一步按层次划分，这样每个组织内的不同用户能够共享该组织队列所分配的资源。在一个队列内，使用FIFO调度政策对应用进行调度。单个作业使用的资源不会超过其队列容量然而，如果队列中由多个作业，并且队列资源不够用，如果仍有可用的空闲资源，那么容量调度器可能会被空余的资源分配给队列的作业，哪怕这超出队列容量。这称之为为“弹性队列”。

#### Fair

![FDwZDJ.png](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/FDwZDJ.png)

Fair调度器是一个队列资源分配方式，在整个时间线上，所有的Job平均的获取资源。默认情况下，Fair调度器知识对内存资源做公平的调度和分配。当集群中只有一个任务运行时，那么此任务会占用整个集群的资源。当其他的任务提交后，那些释放的资源就会被分配给新的Job，所以每个任务最终都能够获取几乎一样多的资源。

#### 对比

| 调度器        | FIFO                             | Capcity                                                      | Fair                                                         |
| ------------- | -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 设计目的      | 最简单的调度器，易于理解和上手。 | 多用户的情况下，最大化集群的吞吐和利用率。                   | 多用户的情况下，强调用户公平地贡献资源。                     |
| 队列 组织方式 | d单队列                          | 树状组织队列，无论夫队列还是子队列都会由资源参数限制，子队列地资源限制计算是基于父队列地。应用提交到叶子队列。 | 树状组织队列。但是父队列和子队列没有参数继承关系。父队列地资源限制对于子队列没有影响。应用提交到叶子队列。 |
| 资源限制      | 无                               | 父子队列之间有容量关系。每个队列限制了资源使用了，全局最大资源使用了，最大活跃应用数量。 | 每个叶子队列有最小共享量，最大资源量和最大活跃应用了。用户有最大活跃应用数量地全局被指。 |
| 队列ACL限制   | 可以限制应用提交权限             | 可以限制应用提交权限和队列开关继承，父子队列间地ACL会继承    | 可以限制应用提交权限，父子队列间地ACL会继承。但是由于支持客户端动态创建队列，需要限制默认队列地应用数量。 |
| 队列排序算法  | 无                               | 按照队列地资源使用了最小的优先                               | 可以限制应用提交权限，父子队列间的ACL会继承。但是由于支持客户端动态创建队列，需要限制默认队列的应用数量。目前，还看不到关闭动态创建队列的选项。 |
| 应用选择算法  | 先进先出                         | 先进先出                                                     | 公平排序算法排序                                             |
| 本地优先分配  | 支持                             | 支持                                                         | 支持                                                         |
| 延迟调度      | 不支持                           | 不支持                                                       | 支持                                                         |
| 资源抢占      | 不支持                           | 不支持                                                       | 支持                                                         |

总结：

- FIFO： 最简单的调度器，按照先进先出的方式处理应用。只有一个队列可提交应用，所有的用户提交到这个队列。可以针对这个队列设置ACL，没有应用优先级可以配置。
- Capcity：可以看作是FIFO的多队列版本，每个队列可以限制资源使用量。但是队列间的资源分配 以使用量安培作为依据，使得容量小的队列有竞争优势，集群整体吞较大。延迟调度机制使得应用可以放弃，跨机器或者跨机架的调度机会，争取本地调度。
- Fair：多队列，多用户共享资源。特有的客户端创建队列的特性，使得权限控制不太完美，根据队列设定的最小共享量或者权重等参数，按照比例共享资源。延迟调度机制跟Capcity的牡蛎类似但是实现方式稍微不同，资源抢占特性，是指调度容器能够依据公平共享算法，计算每个队列应得的资源，将超额资源的队列的部分容器释放掉的特性。

## 总结

![FD0qOg.png](https://s1.ax1x.com/2018/12/19/FD0qOg.png)