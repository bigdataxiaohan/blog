---
title: Flink运行架构
date: 2020-05-02 11:34:18
tags: Flink
categories: 大数据
---

## 组件

### JobManager

1. 控制一个应用程序执行的主进程，每个应用程序都会被一个不同的JobManager所控制。
2. JobManager会先接收到应用程序，应用程序包括：作业图(JobGraph)、逻辑数据流图和打包的所有类库和其他资源的Jar包。
3. JobManager会把JobGraph转换成一个物理层面的数据流图，这个图被叫做“执行图”（ExecutionGraph）,包含了所有可以并发执行的任务。
4. JobManager会向资源管理器（ResourceManager）请求执行任务必要的资源，也就是任务管理器上的slot。一旦获取到足够的资源，就会将执行图分发到真正运行的TaskManager上。

### TaskManager

1. 每一个TaskManager都包含了一定数量的插槽（slots）。插槽的数量限制了TaskManager能够执行的任务数量。
2. 启动后，TaskManager回向资源管理器注册它的插槽，收到资源管理器的指令后，TaskManager就会将一个或者多个插槽提供给JobManager调用。JobManager就可以向插槽分配任务(tasks)来执行了。
3. 在执行过程中，一个TaskManager可以跟其他运行同一个应用程序的TaskManager交换数据。

### ResourceManager

1. 负责管理任务管理器(TaskManager)的插槽（slot）,TaskManager插槽是Flink中定义的处理资源单元。
2. Flink为不同的环境和资源管理工具提供了不同的资源管理器，比如yarn，mesos，k8s
3. 当jobManager申请插槽资源时，resourceManager会将有空闲的插槽的TaskManager分配给JobManager。如果ResourceManager没有足够的插槽来满足jobManager的请求，它还可以向资源提供平台发起会话，提供启动TaskManager进程的容器。

### Dispatcher

1. 可以跨作业运行，它为应用提交提供了rest接口。
2. 当一个应用被提交执行时，分发器就会启动并将应用移交给一个jobManager。
3. Dispatcher也会启动一个web UI，用来方便展示和监控作业的执行信息。
4. Dispatcher在架构中可能并不是必须的，这取决于应用提交运行的方式。

## YARN 提交



![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E5%A4%A7%E6%95%B0%E6%8D%AE/Flink/20200120120550.png)



Flink任务提交后，Client向HDFS上传Flink的Jar包和配置，之后向Yarn ResourceManager提交任务，ResourceManager分配Container资源并通知对应的NodeManager启动ApplicationMaster，ApplicationMaster启动后加载Flink的Jar包和配置构建环境，然后启动JobManager，之后ApplicationMaster向ResourceManager申请资源启动TaskManager，ResourceManager分配Container资源后，由ApplicationMaster通知资源所在节点的NodeManager启动TaskManager，NodeManager加载Flink的Jar包和配置构建环境并启动TaskManager，TaskManager启动后向JobManager发送心跳包，并等待JobManager向其分配任务。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E5%A4%A7%E6%95%B0%E6%8D%AE/Flink/20200120112358.png)



当 Flink 集群启动后，首先会启动一个 JobManger 和一个或多个的 TaskManager。Client提交任务给JobManager，JobManager调度任务到各个TaskManager执行，TaskManager将心跳和统计信息汇报给JobManager。TaskManager之间以流的形式进行数据的传输，JobManger,TaskManager,Client均为独立的JVM进程。

### 相关组件

- **Client** 为提交 Job 的客户端，是任务的执行的起点，负责接收用户的程序代码，然后创建数据流，数据流提交给JobManager方便下一步执行，执行后Client可以将结果返回给用户。

- **JobManager**负责调度Job并协调Task 做 checkpoint，在集群中至少会有一个JobManager、可以有多个**JobManager**并行运行并分担职责。，从 Client 处接收到 Job 和 JAR 包等资源后，会生成优化后的执行计划，并以 Task 的单元调度到各个 TaskManager 去执行。

- **TaskManager** 每一个worker(TaskManager)是一个JVM进程，它可能会在独立的线程上执行一个或多个subtask。在启动的时候就设置好了槽位数（Slot），每个 slot 能启动一个 Task，Task 为线程。从 JobManager 处接收需要部署的 Task，部署启动后，与自己的上游建立 Netty 连接，接收数据并处理。TaskManager并不是最细粒度的概念，每个TaskManager像一个容器一样，包含一个多或多个Slot。

- **Slot**是TaskManager资源粒度的划分，每个Slot都有自己独立的内存。所有Slot平均分配TaskManger的内存，比如TaskManager分配给Solt的内存为8G，两个Slot，每个Slot的内存为4G，四个Slot，每个Slot的内存为2G，Slot仅划分内存，不涉及cpu的划分。每个Slot可以运行多个task，而且一个task会以单独的线程来运行。资源slot化意味着一个subtask将不需要跟来自其他job的subtask竞争被管理的内存，取而代之的是它将拥有一定数量的内存储。这里不涉及到CPU的隔离，只涉及到内存的隔离。

  通过调整task slot的数量，允许用户定义subtask之间如何互相隔离。如果一个TaskManager一个slot，那将意味着每个task group运行在独立的JVM中（该JVM可能是通过一个特定的容器启动的），而一个TaskManager多个slot意味着更多的subtask可以共享同一个JVM。而在同一个JVM进程中的task将共享TCP连接（基于多路复用）和心跳消息。它们也可能共享数据集和数据结构，因此这减少了每个task的负载。

  

  ![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E5%A4%A7%E6%95%B0%E6%8D%AE/Flink/20200123122411.png)

  

  **注意**：

  - TaskSlot是静态的概念，是指TaskManager具有的并发执行能力，可以通过参数taskmanager.numberOfTaskSlots进行配置，而并行度parallelism是动态概念，即TaskManager运行程序时实际使用的并发能力，可以通过参数parallelism.default进行配置。也就是说，假设一共有3个TaskManager，每一个TaskManager中的分配3个TaskSlot，也就是每个TaskManager可以接收3个task，一共9个TaskSlot，如果我们设置parallelism.default=1，即运行程序默认的并行度为1，9个TaskSlot只用了1个，有8个空闲，因此，设置合适的并行度才能提高效率。



![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E5%A4%A7%E6%95%B0%E6%8D%AE/Flink/20200123125002.png)





![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E5%A4%A7%E6%95%B0%E6%8D%AE/Flink/20200123125025.png)



### 执行图

Flink 中的执行图可以分成四层：**StreamGraph** -> **JobGraph** -> **ExecutionGraph** -> **物理执行图**。

**StreamGraph**：是根据用户通过 Stream API 编写的代码生成的最初的图。用来表示程序的拓扑结构。

**JobGraph**：StreamGraph经过优化后生成了 JobGraph，提交给 JobManager 的数据结构。主要的优化为，将多个符合条件的节点 chain 在一起作为一个节点，这样可以减少数据在节点之间流动所需要的序列化/反序列化/传输消耗。

**ExecutionGraph**：JobManager 根据 JobGraph 生成ExecutionGraph。ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。

**物理执行图**：JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个TaskManager 上部署 Task 后形成的图，并不是一个具体的数据结构。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E5%A4%A7%E6%95%B0%E6%8D%AE/Flink/20200123131302.png)