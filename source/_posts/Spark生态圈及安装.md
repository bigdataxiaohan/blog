---
title: Spark生态圈及安装
date: 2019-05-26 21:43:03
tags: Spark
categories:  大数据
---

## Spark

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/数据结构/哈希表/20190526214751.png)

2009年由马泰·扎哈里亚在加州伯克利分校的AMPLab实现开发的子项目,经过开源捐给了Apache基金会,最后成为了我们熟悉的Apache Spark,Spark式式由Scala语言实现的专门为大规模数据处理而设计的快速通用的计算引擎,经过多年的发展势头迅猛,当然,Flink的出现,也将打破Spark在流式计算的一些短板.后续会更新FLink相关的学习记录.

Spark生态系统已经发展成为一个包含多个子项目的集合，其中包含SparkSQL、Spark Streaming、GraphX、MLib、SparkR等子项目，Spark是`基于内存计算的大数据并行计算框架`。除了扩展了广泛使用的 MapReduce 计算模型，而且高效地支持更多计算模式，包括交互式查询和流处理。Spark 适用于各种各样原先需要多种不同的分布式平台的场景，包括批处理、迭代算法、交互式查询、流处理。通过在一个统一的框架下支持这些不同的计算，Spark使我们可以简单而低耗地把各种处理流程整合在一起。而这样的组合，在实际的数据分析 过程中是很有意义的。不仅如此，Spark 的这种特性还大大减轻了原先需要对各种平台分别管理的负担。 

大一统的软件栈，各个组件关系密切并且可以相互调用，这种设计有几个好处：

1、软件栈中所有的程序库和高级组件都可以从下层的改进中获益。

2、运行整个软件栈的代价变小了。不需要运 行 5 到10 套独立的软件系统了，一个机构只需要运行一套软件系统即可。系统的部署、维护、测试、支持等大大缩减。

3、能够构建出无缝整合不同处理模型的应用。



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/数据结构/哈希表/20190526215518.png)

### <font color='red'>Spark Core</font>

实现了 Spark 的基本功能，包含任务调度、内存管理、错误恢复、与存储系统 交互等模块。Spark Core 中还包含了对弹性分布式数据集(resilient distributed dataset，简称RDD)的 API 定义。

### <font color='red'>Spark SQL</font>

是 Spark 用来操作结构化数据的程序包。通过 Spark SQL，我们可以使用 SQL 或者 Apache Hive 版本的 SQL 方言(HQL)来查询数据。Spark SQL 支持多种数据源，比如 Hive 表、Parquet 以及 JSON 等。

### <font color='red'>Spark Streaming</font>

是 Spark 提供的对实时数据进行流式计算的组件。提供了用来操作数据流的 API，并且与 Spark Core 中的 RDD API 高度对应。

### <font color='red'>Spark MLlib</font>
提供常见的机器学习(ML)功能的程序库。包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据 导入等额外的支持功能。 

### <font color='red'>集群管理器</font>
Spark 设计为可以高效地在一个计算节点到数千个计算节点之间伸缩计算。为了实现这样的要求，同时获得最大灵活性，Spark 支持在各种集群管理器(cluster manager)上运行，包括 Hadoop YARN、Apache Mesos，以及 Spark 自带的一个简易调度器，叫作独立调度器。 

## 特点

### <font color='red'>快</font>
与Hadoop的MapReduce相比，Spark基于内存的运算要快100倍以上，基于硬盘的运算也要快10倍以上。Spark实现了高效的DAG执行引擎，可以通过基于内存来高效处理数据流。计算的中间结果是存在于内存中的。

### <font color='red'>易用</font>
Spark支持Java、Python和Scala的API，还支持超过80种高级算法，使用户可以快速构建不同的应用。而且Spark支持交互式的Python和Scala的shell，可以非常方便地在这些shell中使用Spark集群来验证解决问题的方法。

### <font color='red'>通用</font>
Spark提供了统一的解决方案。Spark可以用于批处理、交互式查询（Spark SQL）、实时流处理（Spark Streaming）、机器学习（Spark MLlib）和图计算（GraphX）。这些不同类型的处理都可以在同一个应用中无缝使用。Spark统一的解决方案非常具有吸引力，毕竟任何公司都想用统一的平台去处理遇到的问题，减少开发和维护的人力成本和部署平台的物力成本。

### <font color='red'>兼容性</font>
Spark可以非常方便地与其他的开源产品进行融合。比如，Spark可以使用Hadoop的YARN和Apache Mesos作为它的资源管理和调度器，并且可以处理所有Hadoop支持的数据，包括HDFS、HBase和Cassandra等。这对于已经部署Hadoop集群的用户特别重要，因为不需要做任何数据迁移就可以使用Spark的强大处理能力。Spark也可以不依赖于第三方的资源管理和调度器，它实现了Standalone作为其内置的资源管 理和调度框架，这样进一步降低了Spark的使用门槛，使得所有人都可以非常容易地部署和使用Spark。此外，Spark还提供了在EC2上部署Standalone的Spark集群的工具。

## 用户

我们大致把Spark的用例分为两类：数据科学应用和数据处理应用。也就对应的有两种人群：数据科学家和工程师。

### <font color='red'>数据科学任务</font>
主要是数据分析领域，数据科学家要负责分析数据并建模，具备 SQL、统计、预测建模(机器学习)等方面的经验，以及一定的使用 Python、 Matlab 或 R 语言进行编程的能力。

### <font color='red'>数据处理应用</font>
工程师定义为使用 Spark 开发 生产环境中的数据处理应用的软件开发者，通过对接Spark的API实现对处理的处理和转换等任务。

## 集群角色

从物理部署层面上来看，Spark主要分为两种类型的节点，Master节点和Worker节点：Master节点主要运行集群管理器的中心化部分，所承载的作用是分配Application到Worker节点，维护Worker节点，Driver，Application的状态。Worker节点负责具体的业务运行。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/数据结构/哈希表/20190526220332.png)

从Spark程序运行的层面来看，Spark主要分为驱动器节点和执行器节点。

## 运行模式

### Local

 所有计算都运行在一个线程当中，没有任何并行计算，测试学习练习使用。

local[K]: 指定使用几个线程来运行计算，比如local[4]就是运行4个worker线程。通常我们的cpu有几个core，就指定几个线程，最大化利用cpu的计算能力;

`local[*]`: 这种模式直接帮你按照cpu最多cores来设置线程数了。

### Standalone

构建一个由Master+Slave构成的Spark集群，Spark运行在集群中。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/数据结构/哈希表/20190526220740.png)

### YarnSpark

客户端直接连接Yarn；不需要额外构建Spark集群。有yarn-client和yarn-cluster两种模式，

主要区别在于：Driver程序的运行节点。

yarn-client：Driver程序运行在客户端，适用于交互、调试，希望立即看到app的输出。

yarn-cluster：Driver程序运行在由RM（ResourceManager）启动的AP（APPMaster）适用于生产环境。



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/数据结构/哈希表/20190526220903.png)

### Mesos

Spark客户端直接连接Mesos；不需要额外构建Spark集群。国内应用比较少，更多的是运用yarn调度。

## Spark2.X新特性

### 精简的API

1. 统一的DataFrame和DataSet接口。统
2. 统一Scala和Java的DataFrame、Dataset接口，在R和Python中缺乏安全类型,DataFrame成为主要的程序接口。
3. 新增SparkSession入口，SparkSession替代原来的SQLContext和HiveContext作为DataFrame和Dataset的入口函数。SQLContext和HiveContext保持向后兼容。
4. 为SparkSession通过全新的工作流式配置。
5. 更易用、更高效的计算接口。
6. DataSet中的聚合操作有全新的、改进的聚合接口。

### Spark作为编译器

Spark2.0搭载了第二代Tungsten引擎，该引擎根据现代编译器与MPP数据库的理念来构建的，它将这些理念用于数据处理中，其中的主要的思想就是在运行时使用优化的字节码，将整体查询合称为单个函数，不再使用虚拟函数调用，而是利用CPU来注册中间数据。效果得到了很大的提升

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190526222559.png)

### 智能化程度

为了实现Spark更快、更轻松、更智能的目标、Spark2.X再许多模块上都做了更新，比如Structred Streaming 引入了低延迟的连续处理(Continuous Processing)、支持Stream-steam Joins、通过Pandas UDFs的性能提升PySpark、支持4种调度引擎：Kubernets Clusters 、Standalone、YARN、Mesos。

## 安装

上传并解压spark安装包

```shell
tar -zxvf spark-2.1.1-bin-hadoop2.7.tgz -C /opt/module/ #解压到指定目录
mv spark-2.1.1-bin-hadoop2.7/ spark #重命名spark
```

进入spark安装目录下的conf文件夹

```sh
cd spark/conf/
```

修改slave文件，添加work节点

```shell
[hadoop@datanode1 conf]$ vim slaves
datanode1
datanode2
datanode3
```

修改spark-env.sh文件

```shell
[hadoop@datanode1 conf]$ cp spark-env.sh.template spark-env.sh
[hadoop@datanode1 conf]$ vim spark-env.sh
######################                     配置如下                     ######################
# Options for the daemons used in the standalone deploy mode
SPARK_MASTER_HOST=datanode1  #指定Master
SPARK_MASTER_PORT=7077      #指定Master端口
```

分发

```shell
xsync spark/
```

如果遇到这样的问题

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190526224233.png)

我们需要设置一下JAVA_HOME，需要再sbin目录下的spark-config.sh 文件中加入JAVA_HOME的路径

```shell
vim /opt/module/spark/sbin/spark-config.sh 

export JAVA_HOME=/opt/module/jdk1.8.0_162
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190526225637.png)

访问datanode1:8080即可访问

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527115650.png)

## 测试

```shell
 bin/spark-submit \       
--class org.apache.spark.examples.SparkPi \            #主类
--master spark://datanode1:7077 \                      #master
--executor-memory 1G \							    #任务的资源指定内存为1G
--total-executor-cores 2 \							#使用cpu核数			
./examples/jars/spark-examples_2.11-2.1.1.jar \		  #jar包
100												  #蒙特卡罗算法迭代次数
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190526230433.png)

```shell
./bin/spark-submit \ 
--class <main-class> # 应用的启动类 (如 org.apache.spark.examples.SparkPi)
--master <master-url> \ #指定Master的地址
--deploy-mode <deploy-mode> \  #是否发布你的驱动到worker节点(cluster) 或者作为一个本地客户端 (client) (default: client)*
--conf <key>=<value> \ # 任意的Spark配置属性， 格式key=value. 如果值包含空格，可以加引号“key=value” 
... # other options
<application-jar> \ #打包好的应用jar,包含依赖. 这个URL在集群中全局可见。 比如hdfs:// 共享存储系统， 如果是 file:// path， 那么所有的节点的path都包含同样的jar
application-arguments: 传给main()方法的参数
--executor-memory 1G 指定每个executor可用内存为1G
--total-executor-cores 2 指定每个executor使用的cup核数为2个
```

可以粗略的计算出PI大致为

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190526230454.png)

再启动spark shell的时候我们也可以指定Spark的Master如果我们不指定的话，则使用的使local模式

```java
bin/spark-shell \
--master spark://datanode1:7077 \
--executor-memory 1g \
--total-executor-cores 2
```

Spark Shell中已经默认将SparkContext类初始化为对象sc。用户代码如果需要用到，则直接应用sc即可 

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190526231126.png)

因此我们可以

```scala
sc.textFile("file:///opt/module/spark/word.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```

准备数据

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190526232053.png)

由于使分布式启动我们需要把数据同步一下

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190526233811.png)

## JobHistoryServer

修改spark-default.conf.template名称

```shell
[hadoop@datanode1 conf]$  mv spark-defaults.conf.template spark-defaults.conf
#修改下面配置 确保HDFS开启
spark.master                     spark://datanode1:7077
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://datanode1:9000/sparklog
```

<font color='red'>注意：HDFS上的目录需要提前存在。</font>

修改spark-env.sh文件，添加如下配置

```shell
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=4000 
-Dspark.history.retainedApplications=3 
-Dspark.history.fs.logDirectory=hdfs://datanode1:9000/sparklog"
```

启动历史服务器

```shell
sbin/start-history-server.sh
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190526235029.png)

我们再次执行任务

```shell
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://datanode1:7077 \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527000603.png)

任务过程中会出现这样的界面，任务完成后我们可以查看

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527000746.png)

同时在HDFS上也会生成日志

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527000818.png)

## HA高可用

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527001032.png)

1.首先我们要确保zookeeper正常安装并启动(具体参阅本人博客)

```shell
zkstart
```

修改spark-env.sh的配置文件

```shell
#注释以下内容

#SPARK_MASTER_HOST=datanode1
#SPARK_MASTER_PORT=7077

#添加以下内容
export SPARK_DAEMON_JAVA_OPTS="
-Dspark.deploy.recoveryMode=ZOOKEEPER
-Dspark.deploy.zookeeper.url=datanode1,datanode2,datanode3
-Dspark.deploy.zookeeper.dir=/spark"
```

分发配置

```shell
xsync spark-env.sh
```

datanode1节点上启动所有节点

```shell
[hadoop@datanode1 spark]$ sbin/start-all.sh
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527121554.png)

datanode2启动master

```shell
[hadoop@datanode2 spark]$ sbin/start-all.sh
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527121833.png)

sparkHA访问集群

```shell
 /opt/module/spark/bin/spark-shell --master spark://datanode1:7077,datanode2:7077 --executor-memory 1g --total-executor-cores 1
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527122708.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527122807.png)

我们在datanode2节点模拟节点出现故障

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527123554.png)

任务依旧可以执行。

## YARN

修改hadoop配置文件yarn-site.xml,添加如下内容：

```xml
 <!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
      <name>yarn.nodemanager.pmem-check-enabled</name>
      <value>false</value>
</property>
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
 <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
</property>
```

修改spark-env.sh，添加如下配置：

```shell
YARN_CONF_DIR=/opt/module/hadoop/etc/hadoop  
HADOOP_CONF_DIR=/opt/module/hadoop/etc/hadoop 
```

同步以下配置文件(脚本参考Hadoop篇)

```shell
xsync /opt/module/hadoop/etc/hadoop/yarn-site.xml
xsync /opt/module/spark/conf/spark-env.sh
```

启动HDFS和YARN集群

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527131327.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527131423.png)

## IDEA环境配置

spark shell仅在测试和验证我们的程序时使用的较多，在生产环境中，通常会在IDE中编制程序，然后打成jar包，然后提交到集群，最常用的是创建一个Maven项目，利用Maven来管理jar包的依赖。

首先我们先创建一个Maven的父项目

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.hph</groupId>
    <artifactId>spark</artifactId>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>sparkcore</module>
        <module>sparksql</module>
        <module>sparkGraphx</module>
    </modules>

    <!-- 表明当前项目是一个父项目，没有具体代码，只有声明的共有信息 -->
    <packaging>pom</packaging>

    <!-- 声明公有的属性 -->
    <properties>
        <spark.version>2.1.1</spark.version>
        <scala.version>2.11.8</scala.version>
        <log4j.version>1.2.17</log4j.version>
        <slf4j.version>1.7.22</slf4j.version>
    </properties>

    <!-- 声明并引入公有的依赖 -->
    <dependencies>
        <!-- Logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        <!-- Logging End -->
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
            <!--<scope>provided</scope>-->
        </dependency>

    </dependencies>

    <!-- 仅声明公有的依赖 -->
    <dependencyManagement>
        <dependencies>
            <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-core_2.11</artifactId>
                <version>${spark.version}</version>
                <!-- 编译环境能用，运行环境不可用 -->
                <!--<scope>provided</scope>-->
            </dependency>
            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-sql_2.11</artifactId>
                <version>${spark.version}</version>
                <!-- 编译环境能用，运行环境不可用 -->
                <!--<scope>provided</scope>-->
            </dependency>
            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-streaming_2.11</artifactId>
                <version>${spark.version}</version>
                <!--<scope>provided</scope>-->
            </dependency>
            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-streaming_2.11</artifactId>
                <version>${spark.version}</version>
                <!--<scope>provided</scope>-->
            </dependency>

            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-core_2.11</artifactId>
                <version>${spark.version}</version>
                <!--<scope>provided</scope>-->
            </dependency>

            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-graphx_2.11</artifactId>
                <version>${spark.version}</version>
                <!-- 编译环境能用，运行环境不可用 -->
                <!--<scope>provided</scope>-->
            </dependency>



        </dependencies>
    </dependencyManagement>

    <!-- 配置构建信息 -->
    <build>

        <!-- 声明并引入构建的插件 -->
        <plugins>
            <!-- 设置项目的编译版本 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <!-- 用于编译Scala代码到class -->
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.2.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

        </plugins>

        <!-- 仅声明构建的插件 -->
        <pluginManagement>

            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-assembly-plugin</artifactId>
                    <version>3.0.0</version>
                    <executions>
                        <execution>
                            <id>make-assembly</id>
                            <phase>package</phase>
                            <goals>
                                <goal>single</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>

        </pluginManagement>

    </build>

</project>

```

在创建一个Maven的子项目sparkcore，在sparkcore中创建spark-wordcount项目

### sparkcore

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spark</artifactId>
        <groupId>com.hph</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spark-core</artifactId>
    <packaging>pom</packaging>
    <modules>
        <module>spark-wordcount</module>
    </modules>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
        </dependency>
    </dependencies>

</project>
```

#### spark-wordcount

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spark-core</artifactId>
        <groupId>com.hph</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spark-wordcount</artifactId>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.hph.WordCount</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

```scala
package com.hph

import org.apache.spark.{SparkConf, SparkContext}

object WordCount {

  def main(args: Array[String]): Unit = {
    //声明配置
    val sparkConf = new SparkConf().setAppName("WordCount")

    //创建SparkContext
    val sc = new SparkContext(sparkConf)


    //设置日志等级
    sc.setLogLevel("INFO")
    //读取输入的文件路径
    val file = sc.textFile(args(0))
    //对输入的文本信息进行分割压平
    val words = file.flatMap(_.split(" "))
    //对文本信息进行映射成K,1  
    val word2Count = words.map((_, 1))
    //相同的Key相加  
    val result = word2Count.reduceByKey(_ + _)
     //输入存储路径
    result.saveAsTextFile(args(1))
	
     //关闭资源
    sc.stop()

  }
}
```

打包将我们的包更名为wordcunt.jar执行命令

```shell
spark-submit --class com.hph.WordCount --master spark://datanode1:7077 --executor-memory 1G --total-executor-cores 2 spark-wordcount-1.0-SNAPSHOT.jar hdfs://datanode1:9000//input/test.txt  hdfs://datanode1:9000//output/SPARK_WordCount
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527193259.png)

当然这种就打包就比较麻烦因此我们可以尝试以下别的方法来运行以下。

#### 远程运行

```scala
package com.hph

import org.apache.spark.{SparkConf, SparkContext}

object WordCount {

  def main(args: Array[String]): Unit = {
    //配置用户名
    val properties = System.getProperties
    properties.setProperty("HADOOP_USER_NAME", "hadoop")
    //声明配置
    val sparkConf = new SparkConf().setAppName("WordCount").setMaster("spark://datanode1:7077")
      .setJars(List("E:\\spark2\\sparkcore\\spark-wordcount\\target\\spark-wordcount-1.0-SNAPSHOT.jar"))
      .setIfMissing("spark.driver.host", "192.168.1.1")

    //创建SparkContext
    val sc = new SparkContext(sparkConf)


    //设置日志等级
    sc.setLogLevel("INFO")
    //业务处理
    val file = sc.textFile("hdfs://datanode1:9000/input/test.txt")
    val words = file.flatMap(_.split(" "))
    val word2Count = words.map((_, 1))
    val result = word2Count.reduceByKey(_ + _)
    result.saveAsTextFile("hdfs://datanode1:9000/output/Spark_Driver_On_W10")

    sc.stop()

  }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527194605.png)

这个相当于在W10上执行了任务，宿主机Windos当作了Driver。

#### 本地调试

```scala
package com.hph

import org.apache.spark.{SparkConf, SparkContext}

object WordCount {

  def main(args: Array[String]): Unit = {
    //配置用户名
    val properties = System.getProperties
    properties.setProperty("HADOOP_USER_NAME", "hadoop")
    //声明配置
    val sparkConf = new SparkConf().setAppName("WordCount").setMaster("local[*]")
//      .setJars(List("E:\\spark2\\sparkcore\\spark-wordcount\\target\\spark-wordcount-1.0-SNAPSHOT.jar"))
//      .setIfMissing("spark.driver.host", "192.168.1.1")

    //创建SparkContext
    val sc = new SparkContext(sparkConf)


    //设置日志等级
    sc.setLogLevel("INFO")
    //业务处理
    val file = sc.textFile("D:\\input\\words.txt")
    val words = file.flatMap(_.split(" "))
    val word2Count = words.map((_, 1))
    val result = word2Count.reduceByKey(_ + _)
    result.saveAsTextFile("D:\\output\\SPARK_ON_local")

    sc.stop()

  }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527195233.png)

如果你遇到了错误

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527195302.png)

可以尝试以下方法修复

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527195331.png)





