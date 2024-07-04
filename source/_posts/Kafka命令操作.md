---
title: Kafka命令操作
date: 2018-11-14 14:17:06
tags: Kafka
categories: 大数据
---

## 查看当前服务器所有的topic

```shell
[hadoop@datanode1 kafka]$ bin/kafka-topics.sh --zookeeper datanode1:2181 --list
```

## 创建topic

```shell
[hadoop@datanode1 kafka]$ bin/kafka-topics.sh --zookeeper datanode1:2181 --create --replication-factor 3 --partitions 1 --topic first
选项说明：
	--topic 定义topic名
	--replication-factor  定义副本数
	--partitions  定义分区数	
```

##  删除topic

```shell
[hadoop@datanode1 kafka]$ bin/kafka-topics.sh --zookeeper datanode1:2181 --delete --topic first
```

## 创建生产者发送消息

```shell
[hadoop@datanode1 kafka]$ bin/kafka-console-producer.sh --broker-list datanode1:9092 --topic test
```

## 创建消费者接受消息

```shell
[hadoop@datanode2 kafka]$ bin/kafka-console-consumer.sh --zookeeper datanode1:2181 --from-beginning --topic test
--from-beginning：会把first主题中以往所有的数据都读取出来。根据业务场景选择是否
```



## 查看某一个topic的详情

```shell
[hadoop@datanode1 kafka]$ bin/kafka-topics.sh --zookeeper datanode1:2181 --describe --topic test
Topic:test      PartitionCount:1        ReplicationFactor:3     Configs:
Topic: test     Partition: 0    Leader: 2       Replicas: 2,3,1 Isr: 2,3,1

```

