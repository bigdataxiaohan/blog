---
title: Apache Hudi快速体验
date: 2022-08-06 21:36:01
tags: Hudi
categories: Hudi
---
通过官方提供的样例我们可以构建docker镜像前提是已经安装了docker 和docker-compose

```bash
# 下载docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# 查看docker-compose版本
docker-compose --version
```

### 编译Hudi源码

修改maven的镜像源为阿里云的镜像

```xml
<localRepository>/opt/module/apache-maven/repository</localRepository>
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
    <mirror>
      <id>aliyunCentralMaven</id>
      <name>aliyun central maven</name>
      <url>https://maven.aliyun.com/repository/central/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
    <mirror>
      <id>centralMaven</id>
      <name>central maven</name>
      <url>http://mvnrepository.com/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
```

在hudi的项目路径下执行编译命令，编译也使用默认版本即可，如果为0.11版本之后则需要添加

```shell
mvn clean package -Pintegration-tests -DskipTests
```

编译成功

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/数据湖/20220722101020.png)

打包完成后执行setup_demo.sh 拉取docker镜像。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220722110548226.png)



docker ps 查看对应的进程

![image-20220808100120616](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808100120616.png)

### 配置Host映射

```xml
127.0.0.1 adhoc-1
127.0.0.1 adhoc-2
127.0.0.1 namenode
127.0.0.1 datanode1
127.0.0.1 hiveserver
127.0.0.1 hivemetastore
127.0.0.1 kafkabroker
127.0.0.1 sparkmaster
127.0.0.1 zookeeper
118.195.224.194 hudi
```

###  上传第一批数据到kafka

首先 brew install kcat 安装 kcat对数据进行加工处理。

```shell
 cat docker/demo/data/batch_1.json | kcat -b kafkabroker -t stock_ticks -P
```



### 查看Kafka数据

使用命令查看对应的topic数据

```shell
 kcat -C -b kafkabroker -t stock_ticks
```

![image-20220808100919416](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808100919416.png)

### Kafka中数据写入Hudi中

```shell
docker exec -it adhoc-2 /bin/bash
# 使用下面的命令执行Delta-Looper并在HDFS中摄取到Stock_Ticks_cow表中
spark-submit \
  --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer $HUDI_UTILITIES_BUNDLE \
  --table-type COPY_ON_WRITE \
  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
  --source-ordering-field ts  \
  --target-base-path /user/hive/warehouse/stock_ticks_cow \
  --target-table stock_ticks_cow --props /var/demo/config/kafka-source.properties \
  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider
```

![image-20220808114709211](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808114709211.png)

![image-20220808101156661](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808114741272.png)

```shell
 docker exec -it adhoc-2 /bin/bash
# 运行以下spark-subment命令以执行Delta-Fleoms，并在HDFS中摄取到Stock_Ticks_mor表
spark-submit \
  --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer $HUDI_UTILITIES_BUNDLE \
  --table-type MERGE_ON_READ \
  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
  --source-ordering-field ts \
  --target-base-path /user/hive/warehouse/stock_ticks_mor \
  --target-table stock_ticks_mor \
  --props /var/demo/config/kafka-source.properties \
  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
  --disable-compaction
```

![image-20220808101248582](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808114840170.png)

#### hudi数据与hive集成

```shell
docker exec -it adhoc-2 /bin/bash

/var/hoodie/ws/hudi-sync/hudi-hive-sync/run_sync_tool.sh \
  --jdbc-url jdbc:hive2://hiveserver:10000 \
  --user hive \
  --pass hive \
  --partitioned-by dt \
  --base-path /user/hive/warehouse/stock_ticks_cow \
  --database default \
  --table stock_ticks_cow
  

 /var/hoodie/ws/hudi-sync/hudi-hive-sync/run_sync_tool.sh \
  --jdbc-url jdbc:hive2://hiveserver:10000 \
  --user hive \
  --pass hive \
  --partitioned-by dt \
  --base-path /user/hive/warehouse/stock_ticks_mor \
  --database default \
  --table stock_ticks_mor
```

对于hive表 stock_ticks_cow 支持快照查询和增量查询。

生成两张新表 stock_ticks_mor_rt 和 stock_ticks_mor_ro，前者支持Snapshot查询和Incremental查询，提供近实时的数据，后者支持读优化查询。

#### 使用Hive执行查询

运行 hive 查询以查找为股票代码“GOOG”提取的最新时间戳。您会注意到快照（对于 COW 和 MOR _rt 表）和读取优化查询（对于 MOR _ro 表）都给出了相同的值“上午 10:29”，因为 Hudi 为第一批数据创建了 parquet 文件。

```shell
docker exec -it adhoc-2 /bin/bash

beeline -u jdbc:hive2://hiveserver:10000 \
  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
  --hiveconf hive.stats.autogather=false

```

```sql
root@adhoc-2:/opt# beeline -u jdbc:hive2://hiveserver:10000 \
>   --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
>   --hiveconf hive.stats.autogather=false
Connecting to jdbc:hive2://hiveserver:10000
Connected to: Apache Hive (version 2.3.3)
Driver: Hive JDBC (version 1.2.1.spark2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 1.2.1.spark2 by Apache Hive
# List Tables
0: jdbc:hive2://hiveserver:10000>  show tables;
+---------------------+--+
|      tab_name       |
+---------------------+--+
| stock_ticks_cow     |
| stock_ticks_mor_ro  |
| stock_ticks_mor_rt  |
+---------------------+--+
3 rows selected (0.3 seconds)

# Look at partitions that were added
0: jdbc:hive2://hiveserver:10000> show partitions stock_ticks_mor_rt;
+----------------+--+
|   partition    |
+----------------+--+
| dt=2018-08-31  |
+----------------+--+
1 row selected (0.16 seconds)

# COPY-ON-WRITE Queries:
=========================
0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:29:00  |
+---------+----------------------+--+
1 row selected (7.834 seconds)
0: jdbc:hive2://hiveserver:10000>  select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034650860    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808034650860    | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.747 seconds)
# Merge-On-Read Queries:
==========================

Lets run similar queries against M-O-R table. Lets look at both 
ReadOptimized and Snapshot(realtime data) queries supported by M-O-R table

# Run ReadOptimized Query. Notice that the latest timestamp is 10:29
0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:29:00  |
+---------+----------------------+--+
1 row selected (2.19 seconds)
0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:29:00  |
+---------+----------------------+--+
1 row selected (2.106 seconds)
0: jdbc:hive2://hiveserver:10000>  select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808034809720    | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.607 seconds)
0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808034809720    | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |
+----------------------+---------+----------------------+---------+------------+-----------+--+

```



#### 使用Spark 查询

```scala
docker exec -it adhoc-1 /bin/bash

$SPARK_INSTALL/bin/spark-shell \
  --jars $HUDI_SPARK_BUNDLE \
  --master local[2] \
  --driver-class-path $HADOOP_CONF_DIR \
  --conf spark.sql.hive.convertMetastoreParquet=false \
  --deploy-mode client \
  --driver-memory 1G \
  --executor-memory 3G \
  --num-executors 1


22/07/25 07:46:00 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://adhoc-1:4040
Spark context available as 'sc' (master = local[2], app id = local-1658735167793).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.4
      /_/
         
Using Scala version 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_212)
Type in expressions to have them evaluated.
Type :help for more information.



scala> spark.sql("show tables").show(100, false)
+--------+------------------+-----------+
|database|tableName         |isTemporary|
+--------+------------------+-----------+
|default |stock_ticks_cow   |false      |
|default |stock_ticks_mor_ro|false      |
|default |stock_ticks_mor_rt|false      |
+--------+------------------+-----------+

# Copy-On-Write Table

## Run max timestamp query against COW table

scala> spark.sql("select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+                                                    
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:29:00|
+------+-------------------+

## Projection Query

scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034650860  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808034650860  |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|
+-------------------+------+-------------------+------+---------+--------+

# Merge-On-Read Queries:
==========================

Lets run similar queries against M-O-R table. Lets look at both
ReadOptimized and Snapshot queries supported by M-O-R table

scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:29:00|
+------+-------------------+

# Run Snapshot Query. Notice that the latest timestamp is again 10:29

scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:29:00|
+------+-------------------+


# Run Read Optimized and Snapshot project queries

scala>  spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808034809720  |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|
+-------------------+------+-------------------+------+---------+--------+



scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808034809720  |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|
+-------------------+------+-------------------+------+---------+--------+

```



#### 使用Presto查询

```shell
docker exec -it presto-worker-1 presto --server presto-coordinator-1:8090


[root@biodata bin]# docker exec -it presto-worker-1 presto --server presto-coordinator-1:8090

presto> show catalogs;
  Catalog  
-----------
 hive      
 jmx       
 localfile 
 system    
(4 rows)

Query 20220808_035654_00000_hq6qp, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
0:02 [0 rows, 0B] [0 rows/s, 0B/s]

presto>  use hive.default;
USE
presto:default>  show tables;
       Table        
--------------------
 stock_ticks_cow    
 stock_ticks_mor_ro 
 stock_ticks_mor_rt 
(3 rows)

Query 20220808_035711_00002_hq6qp, FINISHED, 2 nodes
Splits: 19 total, 19 done (100.00%)
0:01 [3 rows, 102B] [2 rows/s, 76B/s]

presto:default> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:29:00 
(1 row)

Query 20220808_035719_00003_hq6qp, FINISHED, 1 node
Splits: 49 total, 49 done (100.00%)
0:04 [197 rows, 426KB] [45 rows/s, 97.8KB/s]

presto:default>  select "_hoodie_commit_time", symbol, ts, volume, open, close from stock_ticks_cow where symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034650860   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808034650860   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 
(2 rows)

Query 20220808_035735_00004_hq6qp, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [197 rows, 429KB] [275 rows/s, 600KB/s]

presto:default>  select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:29:00 
(1 row)

Query 20220808_035742_00005_hq6qp, FINISHED, 1 node
Splits: 49 total, 49 done (100.00%)
0:01 [197 rows, 426KB] [293 rows/s, 636KB/s]

presto:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034809720   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808034809720   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 
(2 rows)

Query 20220808_035747_00006_hq6qp, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [197 rows, 429KB] [343 rows/s, 748KB/s]

```

#### 使用Trino查询

这是使用Trino的查询，目前Trino不支持快照和增量查询在hudi表上

```shell
docker exec -it adhoc-2 trino --server trino-coordinator-1:8091
trino>  show catalogs;
 Catalog 
---------
 hive    
 system  
(2 rows)

Query 20220808_035852_00000_vxj2r, FINISHED, 1 node
Splits: 7 total, 7 done (100.00%)
1.83 [0 rows, 0B] [0 rows/s, 0B/s]

trino>  use hive.default;
USE
trino:default> show tables;
       Table        
--------------------
 stock_ticks_cow    
 stock_ticks_mor_ro 
 stock_ticks_mor_rt 
(3 rows)

Query 20220808_035903_00003_vxj2r, FINISHED, 2 nodes
Splits: 7 total, 7 done (100.00%)
1.91 [3 rows, 102B] [1 rows/s, 53B/s]

trino:default>  select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:29:00 
(1 row)

Query 20220808_035910_00005_vxj2r, FINISHED, 1 node
Splits: 13 total, 13 done (100.00%)
4.77 [197 rows, 442KB] [41 rows/s, 92.7KB/s]

trino:default> select "_hoodie_commit_time", symbol, ts, volume, open, close from stock_ticks_cow where symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034650860   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808034650860   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 
(2 rows)

Query 20220808_035916_00006_vxj2r, FINISHED, 1 node
Splits: 5 total, 5 done (100.00%)
0.67 [197 rows, 450KB] [292 rows/s, 668KB/s]

trino:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:29:00 
(1 row)

Query 20220808_035923_00007_vxj2r, FINISHED, 1 node
Splits: 13 total, 13 done (100.00%)
0.60 [197 rows, 442KB] [328 rows/s, 737KB/s]

trino:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034809720   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808034809720   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 
(2 rows)

Query 20220808_035928_00008_vxj2r, FINISHED, 1 node
Splits: 5 total, 5 done (100.00%)
0.46 [197 rows, 450KB] [428 rows/s, 978KB/s]
```

### 上传第二批数据

使用spark  delta-streamer 上传这批数据，这批数据没有产生新的分区不需要使用ihve-sync来同步数据

```shell
cat docker/demo/data/batch_2.json | kcat -b kafkabroker -t stock_ticks -P
docker exec -it adhoc-2 /bin/bash
```

使用spark-submit   delta-streamer  提取数据到HDFS上的stock_ticks_cow

```shell
spark-submit \
  --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer $HUDI_UTILITIES_BUNDLE \
  --table-type COPY_ON_WRITE \
  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
  --source-ordering-field ts \
  --target-base-path /user/hive/warehouse/stock_ticks_cow \
  --target-table stock_ticks_cow \
  --props /var/demo/config/kafka-source.properties \
  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider
```

使用spark-submit   delta-streamer  提取数据到HDFS上的stock_ticks_mor

```scala
spark-submit \
  --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer $HUDI_UTILITIES_BUNDLE \
  --table-type MERGE_ON_READ \
  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
  --source-ordering-field ts \
  --target-base-path /user/hive/warehouse/stock_ticks_mor \
  --target-table stock_ticks_mor \
  --props /var/demo/config/kafka-source.properties \
  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
  --disable-compaction
```

生成的数据 cow表中生成新版本的parquet文件

 Merge-On-Read table表中，第二批新增的数据生成了尚未合并的delta(log)文件

![image-20220808133212806](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808133212806.png)

使用 Copy-On-Write 表，一旦批次提交，快照查询会立即将更改视为第二批的一部分，因为每次摄取都会创建更新版本的 parquet 文件。

使用 Merge-On-Read 表，第二次摄取只是将批处理附加到未合并的增量（日志）文件中。这是 ReadOptimized 和 Snapshot 查询将提供不同结果的时候。ReadOptimized 查询仍将返回“10:29 am”，因为它只会从 Parquet 文件中读取。快照查询将进行即时合并并返回最新提交的数据，即“上午 10:59”。

#### 使用HIve查询

```shell
docker exec -it adhoc-2 /bin/bash
beeline -u jdbc:hive2://hiveserver:10000 \
  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
  --hiveconf hive.stats.autogather=false

Connecting to jdbc:hive2://hiveserver:10000
Connected to: Apache Hive (version 2.3.3)
Driver: Hive JDBC (version 1.2.1.spark2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 1.2.1.spark2 by Apache Hive
0: jdbc:hive2://hiveserver:10000>  select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
# Copy On Write Table:
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:59:00  |
+---------+----------------------+--+
1 row selected (2.217 seconds)
0: jdbc:hive2://hiveserver:10000>  select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220725021154599    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220725022945766    | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.659 seconds)
As you can notice, the above queries now reflect the changes that came as part of ingesting second batch.

# Merge On Read Table:
# Read Optimized Query

0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:29:00  |
+---------+----------------------+--+
1 row selected (2.055 seconds)
0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808034809720    | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.692 seconds)
# Snapshot Query

0: jdbc:hive2://hiveserver:10000>  select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:59:00  |
+---------+----------------------+--+
1 row selected (1.978 seconds)
0: jdbc:hive2://hiveserver:10000>  select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808040149926    | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.599 seconds)
```

#### 使用Spark查询

```scala
root@adhoc-1:/opt# $SPARK_INSTALL/bin/spark-shell \
>   --jars $HUDI_SPARK_BUNDLE \
>   --driver-class-path $HADOOP_CONF_DIR \
>   --conf spark.sql.hive.convertMetastoreParquet=false \
>   --deploy-mode client \
>   --driver-memory 1G \
>   --master local[2] \
>   --executor-memory 3G \
>   --num-executors 1
22/07/25 03:03:57 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://adhoc-1:4040
Spark context available as 'sc' (master = local[2], app id = local-1658718244614).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.4
      /_/
         
Using Scala version 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_212)
Type in expressions to have them evaluated.
Type :help for more information.

// Copy On Write Table:
scala>  spark.sql("select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+                                                    
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:59:00|
+------+-------------------+



// As you can notice, the above queries now reflect the changes that came as part of ingesting second batch.
// Merge On Read Table:
// Read Optimized Query
scala>  spark.sql("select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:59:00|
+------+-------------------+


scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034650860  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808040120355  |GOOG  |2018-08-31 10:59:00|9021  |1227.1993|1227.215|
+-------------------+------+-------------------+------+---------+--------+
As you can notice, the above queries now reflect the changes that came as part of ingesting second batch.
// Merge On Read Table:
// Read Optimized Query

scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:29:00|
+------+-------------------+

scala>  spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808034809720  |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|
+-------------------+------+-------------------+------+---------+--------+
// Snapshot Query
scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:59:00|
+------+-------------------+

scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808040149926  |GOOG  |2018-08-31 10:59:00|9021  |1227.1993|1227.215|
+-------------------+------+-------------------+------+---------+--------+
```

#### 使用Presto查询

运行同样的查询语句在ReadOptimized上

```sql
root@biodata ~]# docker exec -it presto-worker-1 presto --server presto-coordinator-1:8090

presto> use hive.default;
USE
presto:default> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:59:00 
(1 row)

Query 20220808_054014_00011_hq6qp, FINISHED, 1 node
Splits: 49 total, 49 done (100.00%)
0:01 [197 rows, 426KB] [225 rows/s, 488KB/s]

presto:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034650860   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808040120355   | GOOG   | 2018-08-31 10:59:00 |   9021 | 1227.1993 | 1227.215 
(2 rows)

Query 20220808_054026_00012_hq6qp, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
484ms [197 rows, 429KB] [406 rows/s, 885KB/s]

presto:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:29:00 
(1 row)

Query 20220808_054038_00013_hq6qp, FINISHED, 1 node
Splits: 49 total, 49 done (100.00%)
0:01 [197 rows, 426KB] [316 rows/s, 685KB/s]

presto:default>  select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034809720   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808034809720   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 
(2 rows)

Query 20220808_054043_00014_hq6qp, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
450ms [197 rows, 429KB] [438 rows/s, 954KB/s]
```

#### 使用Trino查询

```sql
root@biodata ~]# docker exec -it adhoc-2 trino --server trino-coordinator-1:8091

trino>  use hive.default;
USE
trino:default>  select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:59:00 
(1 row)

Query 20220808_054140_00012_vxj2r, FINISHED, 1 node
Splits: 13 total, 13 done (100.00%)
0.47 [197 rows, 442KB] [423 rows/s, 951KB/s]

trino:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034650860   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808040120355   | GOOG   | 2018-08-31 10:59:00 |   9021 | 1227.1993 | 1227.215 
(2 rows)

Query 20220808_054150_00013_vxj2r, FINISHED, 1 node
Splits: 5 total, 5 done (100.00%)
0.38 [197 rows, 450KB] [517 rows/s, 1.15MB/s]

trino:default>  select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:29:00 
(1 row)

Query 20220808_054156_00014_vxj2r, FINISHED, 1 node
Splits: 13 total, 13 done (100.00%)
0.43 [197 rows, 442KB] [460 rows/s, 1.01MB/s]

trino:default>  select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034809720   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808034809720   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 
(2 rows)

Query 20220808_054201_00015_vxj2r, FINISHED, 1 node
Splits: 5 total, 5 done (100.00%)
0.38 [197 rows, 450KB] [523 rows/s, 1.17MB/s]
```

#### 增量查询COW表

```shell
docker exec -it adhoc-2 /bin/bash
beeline -u jdbc:hive2://hiveserver:10000 \
  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
  --hiveconf hive.stats.autogather=false

Connecting to jdbc:hive2://hiveserver:10000
Connected to: Apache Hive (version 2.3.3)
Driver: Hive JDBC (version 1.2.1.spark2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 1.2.1.spark2 by Apache Hive
0: jdbc:hive2://hiveserver:10000>  select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034650860    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808040120355    | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.606 seconds)
```

上面的查询查到了条cimmit记录一个是20220808034650860，另一个是20220808040120355 在时间线上。

为了展示增量查询的效果，我们假设读者已经将这些更改视为摄取第一批的一部分。现在，为了让读者看到第二批的效果，他/她必须将开始时间戳保持到第一批的提交时间（20180924064621）并运行增量查询

Hudi 增量模式通过使用 hudi 管理的元数据过滤掉没有任何候选行的文件，为增量查询提供高效扫描。使用上述设置，从提交 20220808034650860 中没有任何更新的文件 ID 将被过滤掉而不进行扫描。这是增量查询：

```sql
0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG' and `_hoodie_commit_time` > '20220808034650860';
```

#### 使用Spark SQL进行增量查询

```scala
docker exec -it adhoc-1 /bin/bash
root@adhoc-1:/opt# 
$SPARK_INSTALL/bin/spark-shell \
   --jars $HUDI_SPARK_BUNDLE \
   --driver-class-path $HADOOP_CONF_DIR \
   --conf spark.sql.hive.convertMetastoreParquet=false \
   --deploy-mode client \
   --driver-memory 1G \
   --master local[2] \
   --executor-memory 3G \
   --num-executors 1

22/07/25 05:41:53 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://adhoc-1:4040
Spark context available as 'sc' (master = local[2], app id = local-1658727720839).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.4
      /_/
         
Using Scala version 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_212)
Type in expressions to have them evaluated.
Type :help for more information.


import org.apache.hudi.DataSourceReadOptions


val hoodieIncViewDF =  spark.read.format("org.apache.hudi").option(DataSourceReadOptions.QUERY_TYPE_OPT_KEY, DataSourceReadOptions.QUERY_TYPE_INCREMENTAL_OPT_VAL).option(DataSourceReadOptions.BEGIN_INSTANTTIME_OPT_KEY, "20220808034650860").load("/user/hive/warehouse/stock_ticks_cow")



hoodieIncViewDF.registerTempTable("stock_ticks_cow_incr_tmp1")
spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow_incr_tmp1 where  symbol = 'GOOG'").show(100, false);

```

#### 手工完成合并文件

```sql
docker exec -it adhoc-1 /bin/bash
/var/hoodie/ws/hudi-cli/hudi-cli.sh

===================================================================
*         ___                          ___                        *
*        /\__\          ___           /\  \           ___         *
*       / /  /         /\__\         /  \  \         /\  \        *
*      / /__/         / /  /        / /\ \  \        \ \  \       *
*     /  \  \ ___    / /  /        / /  \ \__\       /  \__\      *
*    / /\ \  /\__\  / /__/  ___   / /__/ \ |__|     / /\/__/      *
*    \/  \ \/ /  /  \ \  \ /\__\  \ \  \ / /  /  /\/ /  /         *
*         \  /  /    \ \  / /  /   \ \  / /  /   \  /__/          *
*         / /  /      \ \/ /  /     \ \/ /  /     \ \__\          *
*        / /  /        \  /  /       \  /  /       \/__/          *
*        \/__/          \/__/         \/__/    Apache Hudi CLI    *
*                                                                 *
===================================================================

Welcome to Apache Hudi CLI. Please type help if you are looking for help. 
hudi->connect --path /user/hive/warehouse/stock_ticks_mor
Metadata for table stock_ticks_mor loaded
hudi:stock_ticks_mor->
hudi:stock_ticks_mor->compactions show all
╔═════════════════════════╤═══════════╤═══════════════════════════════╗
║ Compaction Instant Time │ State     │ Total FileIds to be Compacted ║
╠═════════════════════════╪═══════════╪═══════════════════════════════╣
║ 20220808055239150       │ REQUESTED │ 1                             ║
╚═════════════════════════╧═══════════╧═══════════════════════════════╝

hudi:stock_ticks_mor->
hudi:stock_ticks_mor->compaction schedule --hoodieConfigs hoodie.compact.inline.max.delta.commits=1
22/08/08 06:19:09 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
22/08/08 06:19:11 WARN DFSPropertiesConfiguration: Cannot find HUDI_CONF_DIR, please set it as the dir of hudi-defaults.conf
22/08/08 06:19:11 WARN DFSPropertiesConfiguration: Properties file file:/etc/hudi/conf/hudi-defaults.conf not found. Ignoring to load props file
22/08/08 06:19:19 WARN HoodieCompactor: After filtering, Nothing to compact for /user/hive/warehouse/stock_ticks_mor
Attempted to schedule compaction for 20220808061907889
hudi:stock_ticks_mor->
hudi:stock_ticks_mor->compactions show all
╔═════════════════════════╤═══════════╤═══════════════════════════════╗
║ Compaction Instant Time │ State     │ Total FileIds to be Compacted ║
╠═════════════════════════╪═══════════╪═══════════════════════════════╣
║ 20220808055239150       │ REQUESTED │ 1                             ║
╚═════════════════════════╧═══════════╧═══════════════════════════════╝

hudi:stock_ticks_mor->
hudi:stock_ticks_mor->compaction schedule --hoodieConfigs hoodie.compact.inline.max.delta.commits=1
22/08/08 06:19:34 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
22/08/08 06:19:36 WARN DFSPropertiesConfiguration: Cannot find HUDI_CONF_DIR, please set it as the dir of hudi-defaults.conf
22/08/08 06:19:36 WARN DFSPropertiesConfiguration: Properties file file:/etc/hudi/conf/hudi-defaults.conf not found. Ignoring to load props file
22/08/08 06:19:44 WARN HoodieCompactor: After filtering, Nothing to compact for /user/hive/warehouse/stock_ticks_mor
Attempted to schedule compaction for 20220808061933685
hudi:stock_ticks_mor->
hudi:stock_ticks_mor->refresh
Metadata for table stock_ticks_mor refreshed.
hudi:stock_ticks_mor->compactions show all
╔═════════════════════════╤═══════════╤═══════════════════════════════╗
║ Compaction Instant Time │ State     │ Total FileIds to be Compacted ║
╠═════════════════════════╪═══════════╪═══════════════════════════════╣
║ 20220808055239150       │ REQUESTED │ 1                             ║
╚═════════════════════════╧═══════════╧═══════════════════════════════╝

# Schedule a compaction. This will use Spark Launcher to schedule compaction
hudi:stock_ticks_mor->compaction run --compactionInstant  20220808055239150 --parallelism 2 --sparkMemory 1G  --schemaFilePath /var/demo/config/schema.avsc --retry 1  
22/08/08 06:20:06 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

22/08/08 06:20:08 WARN DFSPropertiesConfiguration: Cannot find HUDI_CONF_DIR, please set it as the dir of hudi-defaults.conf
22/08/08 06:20:08 WARN DFSPropertiesConfiguration: Properties file file:/etc/hudi/conf/hudi-defaults.conf not found. Ignoring to load props file
00:04  WARN: Timeline-server-based markers are not supported for HDFS: base path /user/hive/warehouse/stock_ticks_mor.  Falling back to direct markers.
00:05  WARN: Timeline-server-based markers are not supported for HDFS: base path /user/hive/warehouse/stock_ticks_mor.  Falling back to direct markers.
00:07  WARN: Timeline-server-based markers are not supported for HDFS: base path /user/hive/warehouse/stock_ticks_mor.  Falling back to direct markers.
Compaction successfully completed for 20220808055239150
# Now refresh and check again. You will see that there is a new compaction requested
hudi:stock_ticks_mor->refresh
Metadata for table stock_ticks_mor refreshed.
hudi:stock_ticks_mor->compactions show all
╔═════════════════════════╤═══════════╤═══════════════════════════════╗
║ Compaction Instant Time │ State     │ Total FileIds to be Compacted ║
╠═════════════════════════╪═══════════╪═══════════════════════════════╣
║ 20220808055239150       │ COMPLETED │ 1                             ║
╚═════════════════════════╧═══════════╧═══════════════════════════════╝
```

#### 使用Hive进行增量查询

```sql
docker exec -it adhoc-2 /bin/bash

beeline -u jdbc:hive2://hiveserver:10000 \
  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
  --hiveconf hive.stats.autogather=false
  
# Read Optimized Query
root@adhoc-2:/opt# beeline -u jdbc:hive2://hiveserver:10000 \
>   --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
>   --hiveconf hive.stats.autogather=false
Connecting to jdbc:hive2://hiveserver:10000
Connected to: Apache Hive (version 2.3.3)
Driver: Hive JDBC (version 1.2.1.spark2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 1.2.1.spark2 by Apache Hive
0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:29:00  |
+---------+----------------------+--+
1 row selected (2.097 seconds)
0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808034809720    | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.707 seconds)

# Snapshot Query
0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:59:00  |
+---------+----------------------+--+
1 row selected (1.967 seconds)
0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808040149926    | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.569 seconds)
# Incremental Query:
set hoodie.stock_ticks_mor.consume.mode=INCREMENTAL;
set hoodie.stock_ticks_mor.consume.max.commits=3;
set hoodie.stock_ticks_mor.consume.start.timestamp=20220808034809720;

select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG' and `_hoodie_commit_time` > '20220808034809720';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808040149926    | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
  
```

#### 读优化和快照查询 MOR使用Spark

```scala
docker exec -it adhoc-1 /bin/bash
$SPARK_INSTALL/bin/spark-shell \
  --jars $HUDI_SPARK_BUNDLE \
  --driver-class-path $HADOOP_CONF_DIR \
  --conf spark.sql.hive.convertMetastoreParquet=false \
  --deploy-mode client \
  --driver-memory 1G \
  --master local[2] \
  --executor-memory 3G \
  --num-executors 1

# Read Optimized Query
scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+                                                    
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:59:00|
+------+-------------------+
scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808040149926  |GOOG  |2018-08-31 10:59:00|9021  |1227.1993|1227.215|
+-------------------+------+-------------------+------+---------+--------+
# Snapshot Query
scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:59:00|
+------+-------------------+
scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808040149926  |GOOG  |2018-08-31 10:59:00|9021  |1227.1993|1227.215|
+-------------------+------+-------------------+------+---------+--------+
```

#### Presto 读优化查询MOR表

```sql



presto> use hive.default;
USE

# Read Optimized Query
presto:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:59:00 
(1 row)

Query 20220808_064105_00016_hq6qp, FINISHED, 1 node
Splits: 49 total, 49 done (100.00%)
0:01 [197 rows, 426KB] [300 rows/s, 651KB/s]

presto:default>  select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034809720   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808040149926   | GOOG   | 2018-08-31 10:59:00 |   9021 | 1227.1993 | 1227.215 
(2 rows)

Query 20220808_064117_00017_hq6qp, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
469ms [197 rows, 429KB] [420 rows/s, 915KB/s]

```



