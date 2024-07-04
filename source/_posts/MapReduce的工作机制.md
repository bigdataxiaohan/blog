---
title: MapReduce的工作机制
date: 2018-12-24 08:49:40
tags: MapReduce
categories: 大数据
---

## 框架

Hadoop2.x引入了一种新的执行机制MapRedcue 2。这种新的机制建议在Yarn的系统上，目前用于执行的框架可以通过mapreduce.framework.name属性进行设置，值“local“表示本地作业运行器，“classic”值是经典的MapReduce框架(也称MapReduce1，它使用一个jobtracker和多个tasktracker)，yarn表示新的框架。

## MR工作运行机制

Hadoop的运行工作机制需要下面5个独立实体：

客户端，提交Mapreduce工作。

 YARN资源管理器（ResourceManager），负责协调集群上计算机资源的分配。

YARN节点管理(NodeManager)，负责启动和监视集群中机器上的计算容器(container)。

MapReduce的application master（简写为MRAppMaster），负责协调运行MapReduce的作业，它 和MapReduce任务在容器中运行，这些容器由资源管理器节点管理器进行管理。

分布式文件系统（一般为HDFS），用来与其他实体间共享作业文件。

![Fc1h8S.png](https://s1.ax1x.com/2018/12/25/Fc1h8S.png)

 

### 作业提交

可以通过一个简单的方法调用来运行MapReduce作业：Job对象的submit()方法。注意也可以调用waitForCompletion()，它用于提交以前没有提交过的作业，并等待它的完成。submit()方法调用了封装了大量的处理细节。Job的submit()方法创建了一个内部的JobSummiter实例，并且调用其submitJobInternal()方法（参见上图步骤1），提交作业后，waitForCompletion()每秒轮询作业进度，如果发现自己上次报告由改变，便把进度报告到控制台。作业完成后，如果成功，就是显示出计数器；如果失败，则导致作业失败的错误会记录到控制台。

JobSummiter 所实现的作业提交过程如下所述：

- Client 向资源提管理器请求一个新应用ID，用于MapReduce作业ID。（参见步骤2）
- 检查作业的输出说明，例如：如果没有指定目录或者输出目录已经存在，作业就不提交，错误抛回给MapReduce程序。
- 计算作业的输入分片。如果分片无法计算，比如输入路径不存在，作业就不提交，错误返回MapReduce程序 。
- 将运行作业所需要的资源（包括作业Jar文件、配置文件和计算所得的输入分片）复制到一个以作业ID命名的HDFS目录中(请参见步骤3a)。作业Jar的副本较多(由mapreduce.client.submit.file.relication属性控制。默认值：10)，因此在运行作业的任务时，集群中有很多个副本可供节点管理器访问。保存成功后，Client即可以从HDFS获取文件原信息(步骤3b)。
- 通过调用资源管理器的submitApplication()方法提交作业。参见步骤4。

### 作业初始化

资源管理器收到调用它的submitApplication()消息后，便将请求传递给YARN调度器（scheduler）。调度器分配一个容器，然后资源管理器在节点管理器的管理下在容器中启动application master的进程。（步骤5a和5b）。

MapReduce作业的application master是一个Java应用程序，它的主类是MRAppMaster。由于接接收来自任务的进度和完成报告（步骤6），因此application master对作业的初始化是通过创建多个簿记对象以保持对作业进度的追踪来完成的。接下来，它接收来自共享文件系统的、在客户端计算的输入分片（步骤7）。然后对每一个分片创建一个map对象以及由mapreduce.job.reduces属性（通过作业的setNumReduceTask()方法设置）确定的多个reduce任务对象。任务 ID（Task ID）在此时分配。

application master 必须决定如何运行构成MapReduce作业的各个任务。如果作业很小，就选择和自己在同一个JVM上运行任务。在一个节点顺序运行这些任务相比，当application master 判断在新的容器中分配和运行任务的开销大于并行运行它们的开销时，就会发生这一情况。这样的小作业称为uber任务。

小作业：默认情况下小作业就是少于mapper且只有一个输入大小小于一个HDFS块的作业(通过设置mapreduce.job.ubertask.maxmaps)和mapreduce.job.ubertask.maxbytes可以改变这几个值）。必须明确启动uber任务（对于单个作业，或者是对整个集群），具体方案是将mapreduce.job.ubertask.enable设置成true。

最后，在任何任务运行之前，application master调用setupJob()方法设置OutputCmmitter。FileOutputCmmitter为默认值，表示将建立作业的最终输出目录及任务输出的临时作业空间。

### 任务分配

如果作业不合适作为uber任务运行，那么applicationmaster就会为该作业中所有map任务和reduce任务请求容器（步骤8）.首先为Map任务发出请求，该请求优先级要高于reduce的任务请求，这是因为所有的map任务须在reduce的排序阶段能够启动（shuffle）。直到有5%的map任务已经完成时，为reduce任务的请求才会发出。

reduce任务能够在集群中任意位置运行，但是map的任务请求有着数据本地化局限，这也是任务调度器所关注的，在理想的情况下，任务是数据本地化(data local)的，因为这在分片驻留的统一节点上运行。可选的情况是，任务可能是机架本地化(rack local)的，意味着任务在分片驻留的统一节点运行。即和分片在同一机架而非同一个节点上运行，有些任务既不是数据本地化的，也不是机架本地化的，他们会从别的机架，而不是运行所在的机架上获取自己的数据。对于一个特定作业运行，可以通过查看作业的计数器来确定每个本地化层次上运行的任务数量。

请求也为任务制定了内存需求和CPU数。默认情况下，每个map任务和reduce任务都分配到1024MB的内存和一个虚拟的内核，这些值可以在每个作业的基础上进行配置：

- mapreduce.map.memory.mb
- mapreduce.reduce.memory.mb
- mapreduce.map.cpu.vcores
- mapreduce.map.cpu.vcores.memory.mb

### 任务执行

一旦资源管理器的调度器为任务分配了一个特定节点上的容器，application master就通过与节点管理器来启动容器（步骤9a 和 9b），该任务由主类为YarnChild的一个Java应用程序执行。在它运行任务之前，首先将任务需要的资源本地化，包括作业的配置、JAR文件和所有来自分布式缓存的文件(步骤10).最后，运行map任务和reduce任务(步骤 11)。

YarnChild在指定的JVM中运行，因此用户定义的map和reduce函数(甚至是YarnChild)中的任何缺陷不会影响到节点管理器。例如导致其崩溃或挂起。

每个任务都能够执行搭建(setup)和提交(commit)动作，他们和任务本身在同一个JVM中运行，并由作业的OutputCommitter确定。对于基于文件的作业，提交动作将任务输出由临时位置搬移到最终位置。提交协议确保当前推测(speculative execution)被启用时，只有一个任务副本被提交，其他的都被取消。

### 进度和状态的更新

MapReduce作业是长时间运行的批量作业，运行时间范围从数秒到数小时。可能是一个很长时间段，所以对于用户而言，能够得知关于作业进展的一些反馈时很重要的。一个作业和他的每个任务都有一个状态(status)，包括：作业或任务的状态(比如，运行中，成功完成，失败)、map和reduce的进度、作业计数器的值、状态消息或描述(可以由用户代码来设置)。这些状态在作业期间不断改变。

任务在运行时，对其他进度(progress 即任务完成百分比) 保持追踪。对map任务，任务进度是已处理输入所占的比例。对于reduce，系统会估计已处理reduce输入的比例。整个部过程分为三部分，与shuffle的三个阶段对应（详情参见shuffle和排序）。比如如果任务已经执行reducer一般的输入，那么任务的进度是5/6,这是因为已经完成复制和排序阶段（每个占1/3），并且已经完成reduce阶段的一半1/6）。

**MapReduce中的进度的组成：进度并不总是可测量的，但是虽然如此，它能告诉Hadoop有个任务正在做一些事情比如:正在些输出记录的任务是由进度的，即使此时这个进度不能用需要写的总量的百分比来表示（因为即便是产生这些输出的任务，也可能不知道需求写的总量。进度报告很重要，构成进度的所有操作如下：**

- 读入一条输入记录(在mapper或reducer中)。
- 写入一条输出记录(在mapper或reducer中)。
- 设状态描述（通过Reporter或者TaskAttemptConetext的setStatus()方法）。
- 增加计数器的值（使用Reporter的incrCounter()方法或Counter的increment()方法）。
- 调用Reporter或TaskAttemptContext的progress()方法。

任务也有一组计数器，负责对任务运行过程中各个事件进行计数，这些技术其要么内置于框架中，列入一些如的map输出记录数，要么由用户自己定义。

当map任务或reduce任务运行时，子进程和自己的父application master通过 umbilical 接口通信。每隔3秒，任务通过这个umbilicali接口想自己的application master报告进度和状态(包括计数器),application master会形成一个作业的汇聚视图(aggregate view)。

资源管理器的界面显示了所有运行中的应用程序，并且分别由链接指应用各自的application master的界面，这些界面展示了MapReduce作业的诸多细节，包括进度。

在作业期间，客户端每秒轮询一次 application master以接收最新的状态(轮询时间间隔通过mapreduce.client.progressmonitor.polinterval设置)。客户端也可以用Job的getStatus()方法得到一个JobStatus的实例，后者包含作业的所有状态信息。

![F64LfP.png](https://s1.ax1x.com/2018/12/24/F64LfP.png)

### 作业完成

当application master收到作业最后一个任务已完成的通知后，便把作业状态设置为“成功”。然后在Job轮询状态时，便一直到任务已经完成，于是Job打印一条消息告知用户，然后从waitForCompletion()方法返回。Job的统计信息和计数值也在这个时候输出到控制台。如果ApplicationMaster有相应的设置，也会发送一个HTTP做作业通知。希望收到回调指令的客户端可以通过mapreduce.job.end-notification.url属性来进行这项设置。

最后，作业完成时，application master和任务管理器清理其工作状态(这样中间输出将被删除)，OutputCommitter的commitJob()方法会被调用，作业信息由历史服务器存档，以便日后用户需要时可以查询。

## Shuffle 与 排序

MapReduce确保每个redcuer的输入都是按键排序，将map的输出作为输入传给reducer的过程称为shuffle。学习shuffle时如何工作，因为它有助于我们了解工作机制方便我们优化MapReduce程序。shuffle属于不断被优化和改进的代码库的一部分，因此下面描述了一些细节（可能会随着版本升级而改变）。从许多方面来看，shuffle是MapReduce的“心脏”，是MapReduce最核心的部分。

### map端

map函数开始产生输出时，并不是简单地将它写到磁盘。这个过程更复杂，它利用缓冲地方式写到内存并出于效率的考虑进行预排序。

![F6jll9.png](https://s1.ax1x.com/2018/12/24/F6jll9.png)

每个map任务都有一个环形内存缓冲区用于存储任务输出。在默认情况下，缓冲区为100M这个值可以通过改变mapreduce.task.io.sort.mb来调整。一旦缓冲区内容达到阈值(mapreduce.map.sort.spill.percent，默认时0.8或者80%)，一个后台线程便开始把内容溢写(split)到磁盘。在溢出些到磁盘的过程中，map输出继续写到缓冲区，但如果在此间缓存被填满，map会被阻塞直到写磁盘过程完成。溢写过程按轮询方式将缓冲区中的内容写道mapreduce.cluster.local.dir属性在作业特定子目录下指定目录中。

在写磁盘之前，线程首先根据数据最终要传的reducer把数据分成相应的分区(partion)。在每个分区中，后台线程案件进行内存中排序，如果有一个combiner函数，它就在排序后的输出上运行。运行combiner函数使得map输出结果更紧凑，因此减少到写到磁盘的数据和传递给reducer的数据。

每次内存缓冲区到达溢出的阈值，就会新建一个一溢出文件(splil file)，因此在map任务写完其最后一个输出记录之后，会有几个溢出文件。在任务完成之前。溢出文件被合并成一个已分区且已排序的输出文件。配置属性mapreduce.task.io.sort.factor控制一次最多能合并多少流，默认值是10。

如果至少存在3个溢出文件(通过mapreduce.combine.minspills属性设置)时，则(combine.minspills属性设置)时，则combiner就会在输出文件写到磁盘之前再次运行，combiner可以在输入上反复运行，但并不影响最终结果。如果只有1或2个溢写文件，那么由于map输出规模减少，因而不值得调用combiner带来的开销，因此不会为该map输出再次运行combiner。

在将压缩map输出写到磁盘的过程中对它进行压缩往往时很好的主意，因为这样会写磁盘的速度更快，节约磁盘空间，并且减少传给reducer的数据量，在默认的情况下输出是不压缩的，但只要将mapreduce.map.output.compress设置为true，就可以启用此功能。

reducer通过HTTP得到输出文件的分区。用于文件分区的工作线程的数量由任务的mapreduce.shuffle.max.threads属性控制，此属性针对的时每一个节点管理器，而不是针对每个map任务，默认值是0将最大线程设置为机器处理器数量的两倍。

### reducer端

map输出文件位于运行map任务的tasktracker的本地磁盘**（注意：尽管map输出经常写道maptasktracker的本地磁盘，但是reduce输出并不这样）**，现在tasktracker需要为分区文件运行reduce任务，并且reduce任务需要集群上若干map任务的map输出作为其特殊的分区文件，每一个map任务的完成时间可能不同，因此在每个任务完成时，reduce任务就开始复制其输出。默认值是5个线程，但是默认值可以通过设置mapreduce.reduce.shuffle.parallelcopies属性设置。

reducer如何知道要从哪台机器取得map输出：

map任务完成后，他们会用心跳机制通知他们的application master。因此对于指定作业，application master知道map输出和主机位置之间的映射关系。reducer中的一个线程定期询问master以便获取map输出主机的位置，直到获得所有输出位置。

由于第一个reducer可能失败，因此主机并没有在第一个reducer检测到输出时就立即从磁盘上删除他们，相反，主机会等待，直到application告诉他删除map的输出，这是作业完成后执行的。

如果map的输出相当小，会被复制到reduce任务的JVM内存中（缓冲区大小由mapreduce.reduce.shuffle.input.buffer.percent属性控制，用于指定此用途的堆空间的百分比），否则，map输出被复制到磁盘。一旦内存缓冲区达到阈值大小（由mapreduce.reduce.merge.percent决定）或达到map输出阈值（由mapreduce.reduce.merge.inmem.threshold控制），则合并后溢出写到磁盘中。如果指定combiner，则在合并期间运行它以降低写入磁盘的数据量。

随着磁盘上副本增多，后台线程会将他们合并为更大的、排好序的文件。这会为后面的合并节省一些时间，注意为了合并，压缩的map输出通过map任务都必须在内存中被解压缩。复制完所有map输出后，reduce任务进入排序阶段(更准确的说时合并阶段，而排序时在map端进行的)，这个阶段将合并map输出，维持其顺序排序。这是循环进行的。比如，如果有50个map输出，而合并因子是10（10默认设置，由mapreduce.task.io.sort.factor属性设置，与map的合并类似），合并将进行5趟。每趟将10个文件合并成一个文件，因此最后有5个中间文件。

在最后阶段，即reduce阶段，直接把数据输入reduce函数，从而省略了一次磁盘往返的形成，并没有将这个5个文件合并成一个已经排序的文件作为最后一趟。最后的合并可以来及内存和磁盘片段。

每趟合并的文件数实际上比实例中展示的有所不同，目标是合并最小数量的文件以便满足最后一趟的合并系数。因此如果有40个文件，我们不会在四趟中每趟合并10个文件从而得到4个文件，相反第一次只合并4个文件随后三趟合并完整的10个文件，在最后一趟中，4个已经合并的文件和与下来的6个（未合并的）文件共合计10个文件如图所述。                   

![FcSwUP.png](https://s1.ax1x.com/2018/12/24/FcSwUP.png)



这合并并没有改变合并次数，它只是一个优化措施，目的是尽量减少写到磁盘的数据量，因为最后一趟总是直接合并到reduce。

在reduce阶段，对已经排序输出中的每个键调用reduce函数。此阶段的输出直接写到输出文件系统，一般为HDFS。如果采用HDFS，由于节点管理器也运行数据节点。所以第一个块副本将被写到本地磁盘文件。

## 配置调优

#### map端

| 属性名称                         | 类型      | 默认值                                     | 说明                                                         |
| -------------------------------- | --------- | ------------------------------------------ | ------------------------------------------------------------ |
| mapreduce.task.io.sort.mb        | int       | 100                                        | 排序map输出是所使用的内存缓存缓冲区的大小，以兆字节为单位。  |
| mapreduce.map.sort.spill.precent | float     | 0.80                                       | map输出内存缓冲和利用开始磁盘一些过程的边界索引，这两者使用比例的阈值。 |
| mapreduce.task.io.sort.factor    | int       | 10                                         | 排序文件时，一次最多合并的流数。这个属性也在reduce中使用，将此值增加到100是很常见的。 |
| mapreduce.map.combine.minspills  | int       | 3                                          | 运行combiner所需最少溢出文件数（如果已经指定combiner）。     |
| mapreduce.map.out.put.compress   | boolean   | fasle                                      | 是否压缩map输出。                                            |
| mapreduce.output.compress.codec  | classname | org.apache.hadoop.io.compress.DefaultCodec | 用于map输出的压缩解码器。                                    |
| mapreduce.shuffle.max.threads    | int       | 0                                          | 每个节点管理器的工作线程数，用于将map输出到reducer。这是集群范围设置，这是集群的范围的设置，不能由单个作业设置，0表示使用Netty默认值，即两倍于可用的处理器数。 |

总的原则是给shuffle过程尽量多提供内存空间。然而一有个平衡问题，也就是要确保map函数和reduce函数能够得到足够的内存来运行。这就是为什么写map函数和reduce函数时尽量少用内存的原因。他们不一般不应该无限使用内存(例如，应避免在map中堆积数据)。

运行map任务和reduce任务的JVM，其内存大小由mapred.child.java.opts属性设置，任务节点上的内存应该尽可能设置的大些。

 在map端，可以通过避免多次溢写磁盘来获得最佳性能；一次最佳的情况。如果能估算map输出大小，就可以合理地设置mapreduce.task.io.sort.*属性来尽可能减少溢写的次数，具体而言，如果可以，就要增加mapreduce.task.io.sort.mb的值。MapReduce设计计数器（“SPILLED_RECORDS”）计算在作业运行整个阶段中溢写磁盘的记录数，对于调优很有帮助。注意，这个计数器包括map和reduce两端的溢出写。

在reduce端，中间数据全部驻留在内存时，就能够获得最佳性能，在默认情况下，这就是不可能发生的，因为所有内存一般都会预留给reduce函数。但如果reduce函数的内存需求不大，把mapreduce.reduce.merge.inmem.threshold设置为0，把mapreduce.reduce.input.buffer.percent设置为1.0或者更低的值。

#### reduce端



| 属性名称                                       | 类型  | 默认值 | 描述                                                         |
| ---------------------------------------------- | ----- | ------ | ------------------------------------------------------------ |
| mapreduce.reduces.shuffle.parallelcopies       | int   | 5      | 用于map输出复制到reduce的线程数。                            |
| mapreduce.reduce.shuffle.maxfetchfailures      | int   | 10     | 在声明之前，reducer获取一个map输出花的最大时间。             |
| mapreduce.task.shuffle.maxfetchfailures        | int   | 10     | 排序文件时一次最多合并流的数量，这个属性也在map端使用        |
| mapreduces.reduce.shuffle.input.buffer.percent | float | 0.70   | 在shuffle的复制阶段，分配给map输出的缓冲区对空间的百分比。   |
| mapreduce.reduce.shuffle.merge.percent         | float | 0.66   | map输出缓冲区(由map.job.shuffle.input.buffer.percent定义)的阈值使用比例。用于启动合并并输出和磁盘溢写出的过程。 |
| mapreduce.reduce.merge.inmem.thresholld        | int   | 1000   | 启动合并输出和磁盘溢写的过程的map输出的阈值数。0或者更小的数意味着没有阈值限制，溢写行为由mapreduce.reduce.shuffle.percent单独控制。 |
| mapreduce.reduce.input.buffer.percent          | float | 0.0    | 在reduce的过程中，在内存中保存map输出的空间占整个堆空间的比例。reduce阶段开始时，内存中的map输出大小不能大于这个值。默认情况下，在reduce任务开始之前，所有的map输出都合并到磁盘上，以便为reducer提供尽可能多的内存，然而，如果reducer需要的内存较少，可以增加此值来最小化访问磁盘的次数。 |

​					

​																	参考资料:《Hadoop权威指南》