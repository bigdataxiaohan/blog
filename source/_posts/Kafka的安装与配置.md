---
title: Kafka和的安装与配置 
date: 2018-11-14 11:55:43
tags: Kafka
categories: 大数据
---

## 集群规划

| datanode1 | datanode2 | datanode3 |
| --------- | --------- | --------- |
| zk        | zk        | zk        |
| kafka     | kafka     | kafka     |



## kafka jar包下载地址

<http://kafka.apache.org/downloads.html>

## kafka集群安装部署 

1.  解压安装包  

```shell
[hadoop@datanode1 software]$ tar -zxvf kafka_2.11-0.8.2.2.tgz -C /opt/module/
```

2. 修改解压后的名称

```shell
[hadoop@datanode1 module]$ mv kafka_2.11-0.8.2.2/ kafka
```

3. /opt/module/kafka目录下创建logs文件夹

```shell
[hadoop@datanode1 kafka]$ mkdir logs/
```

4. 修改配置文件

```shell
[hadoop@datanode1 kafka]$ cd config/
[hadoop@datanode1 config]$ vim server.properties
#broker的全局唯一编号，不能重复
broker.id=0
#删除topic功能使能
delete.topic.enable=true
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘IO的现成数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的缓冲区大小
socket.request.max.bytes=104857600
#kafka运行日志存放的路径	
log.dirs=/opt/module/kafka/logs
#topic在当前broker上的分区个数
num.partitions=1
#用来恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1
#segment文件保留的最长时间，超时将被删除 7天
log.retention.hours=168
#配置连接Zookeeper集群地址
zookeeper.connect=datanode1:2181,datanode2:2181,datanode2:2181

```

 5.配置环境变量

```shell
[hadoop@datanode1 config]$ sudo vim /etc/profile
#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin
[hadoop@datanode1 config]$ source /etc/profile
```

6.分发安装包

```shell
[hadoop@datanode1 module]$ xsync kafka
##分发完毕后要在其他节点上配置/opt/module/kafka/config/server.properties broker.id  值  笔者在这里也是 坑了半天发现分发之后 忘记了改broker.id 值
```

7.启动集群

````shell
[hadoop@datanode1]$ bin/kafka-server-start.sh config/server.properties &
[hadoop@datanode2]$ bin/kafka-server-start.sh config/server.properties &
[hadoop@datanode3]$ bin/kafka-server-start.sh config/server.properties &
````



## Kafaka  Manager

为了简化开发者和服务工程师维护Kafka集群的工作，yahoo构建了一个叫做Kafka管理器的基于Web工具，叫做 Kafka Manager。这个管理工具可以很容易地发现分布在集群中的哪些topic分布不均匀，或者是分区在整个集群分布不均匀的的情况。它支持管理多个集群、选择副本、副本重新分配以及创建Topic。同时，这个管理工具也是一个非常好的可以快速浏览这个集群的工具，有如下功能：

> 1.管理多个kafka集群
> 2.便捷的检查kafka集群状态(topics,brokers,备份分布情况,分区分布情况)
> 3.选择你要运行的副本
> 4.基于当前分区状况进行
> 5.可以选择topic配置并创建topic(0.8.1.1和0.8.2的配置不同)
> 6.删除topic(只支持0.8.2以上的版本并且要在broker配置中设置delete.topic.enable=true)
> 7.Topic list会指明哪些topic被删除（在0.8.2以上版本适用）
> 8.为已存在的topic增加分区
> 9.为已存在的topic更新配置
> 10.在多个topic上批量重分区
> 11.在多个topic上批量重分区(可选partition broker位置)

这里提供编译好了的包，下载后可以直接使用，可以不用去sbt编译。
链接：http://pan.baidu.com/s/1bQ3lkM 密码：c251

将下载完之后的上传到Linux上解压

```shell
[hadoop@datanode1 software]$ unzip kafka-manager-1.3.0.7.zip
[hadoop@datanode1 software]$ mv kafka-manager-1.3.0.7 kafka-manager 
```

修改application.conf中zk配置

```shell
http.port=9001 （默认9000）  #会与hadoop端口发生冲突
kafka-manager.zkhosts="192.168.1.101:2181,192.168.1.102:2181,192.168.1.103:2181" #写ip地址不写主机名
```

用kafkamanage是在jdk8基础上的，所以先安装jdk8,只需下载解压即可。

```shell
#想要看到读取,写入速度需要开启JMX，修改kafka-server-start.sh 添加一行即可：添加JMX端口8999
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then 
export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G" 
export JMX_PORT="8999" 
fi
##如果初始化内存不够 KAFKA_HEAP_OPTS="-Xmx1G -Xms1G" 可以设置小一些 JVM系列涉及到到过,之所以设置相同为了防止内存抖动
```

 想要看到读取,写入速度需要开启JMX，修改kafka-server-start.sh 添加一行即可：添加JMX端口8999

```shell
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"  ## 和你自己的设置内存带下保持相同
export JMX_PORT="8999" 
fi
##"每个kafka broker都需要修改，修改后进行重启kafka"
```

启动kafka manager

```shell
[hadoop@datanode1 module]$ cd kafka
[hadoop@datanode1 module]$ bin/kafka-manager -java-home /opt/module/jdk1.8.0_162/
在后台运行 
[hadoop@datanode1 bin]$ nohup ./kafka-manager  -java-home /opt/module/jdk1.8.0_162/  -Dconfig.file=../conf/application.conf >/dev/null 2>&1 &
```

在localhost:9001查看web页面

创建cluster: 点击顶部Cluster、Add Cluster





配置cluster





​			名字；集群zkhost格式：host1:2181,host2:2181,host3:2181
​			  kafka版本，勾选上JMX和下面的勾选框，可显示更多指标

创建完毕后，可查看



topics相关：



