---

title: Spark 集成Hudi
date: 2022-12-15 18:59:16
tags:tags: Hudi
categories: Hudi
---

### Hudi 支持Spark 版本

| Hudi        | Supported Spark 3 version                        |
| ----------- | ------------------------------------------------ |
| 0.12.x      | 3.3.x，3.2.x，3.1.x                              |
| 0.11.x      | 3.2.x（default build, Spark bundle only），3.1.x |
| 0.10.x      | 3.1.x(default build), 3.0.x                      |
| 0.7.0-0.9.0 | 3.0.x                                            |
| 0.6.0       | and prior	Not supported                       |

### 安装Spark

```shell
wget https://archive.apache.org/dist/spark/spark-3.2.2/spark-3.2.2-bin-hadoop3.2.tgz

tar -zxvf spark-3.2.2-bin-hadoop3.2.tgz -C /opt/module/
mv /opt/module/spark-3.2.2-bin-hadoop3.2 /opt/module/spark-3.2.2
```

### 配置环境变量

```shell
sudo vim /etc/profile.d/my_env.sh

export SPARK_HOME=/opt/module/spark-3.2.2
export PATH=$PATH:$SPARK_HOME/bin

source /etc/profile.d/my_env.sh
```

### 拷贝编译好的jar包到spark jars目录下

```shell
 scp packaging/hudi-spark-bundle/target/hudi-spark3.2-bundle_2.12-0.12.0.jar   root@hadoop102:/opt/module/spark-3.2.2/jars
```

### 启动Spark-shell

```scala
#针对Spark 3.2
spark-shell \
  --conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer' \
  --conf 'spark.sql.catalog.spark_catalog=org.apache.spark.sql.hudi.catalog.HoodieCatalog' \
  --conf 'spark.sql.extensions=org.apache.spark.sql.hudi.HoodieSparkSessionExtension'
```

#### 设置表名，基本路径和数据生成器

```scala
import org.apache.hudi.QuickstartUtils._
import scala.collection.JavaConversions._
import org.apache.spark.sql.SaveMode._
import org.apache.hudi.DataSourceReadOptions._
import org.apache.hudi.DataSourceWriteOptions._
import org.apache.hudi.config.HoodieWriteConfig._

val tableName = "hudi_trips_cow"
val basePath = "file:///tmp/hudi_trips_cow"
val dataGen = new DataGenerator
```

#### 插入数据

```scala
val inserts = convertToStringList(dataGen.generateInserts(10))
val df = spark.read.json(spark.sparkContext.parallelize(inserts, 2))
df.write.format("hudi").
  options(getQuickstartWriteConfigs).
  option(PRECOMBINE_FIELD_OPT_KEY, "ts").
  option(RECORDKEY_FIELD_OPT_KEY, "uuid").
  option(PARTITIONPATH_FIELD_OPT_KEY, "partitionpath").
  option(TABLE_NAME, tableName).
  mode(Overwrite).
  save(basePath)
```

![image-20221215192503146](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221215192503146.png)

#### 查询数据

```scala
val tripsSnapshotDF = spark.
  read.
  format("hudi").
  load(basePath)
tripsSnapshotDF.createOrReplaceTempView("hudi_trips_snapshot")
spark.sql("select fare, begin_lon, begin_lat, ts from  hudi_trips_snapshot where fare > 20.0").show()
```

![image-20221215192619460](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221215192619460.png)

```scala
spark.sql("select _hoodie_commit_time, _hoodie_record_key, _hoodie_partition_path, rider, driver, fare from  hudi_trips_snapshot").show()
```

![image-20221215192741117](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221215192741117.png)

#### 时间录行查询

```scala
spark.read.
  format("hudi").
  option("as.of.instant", "20221215192246898").
  load(basePath).
  show()
  
spark.read.
  format("hudi").
  option("as.of.instant", "2022-12-15 19:22:46.898").
  load(basePath).
  show()
  
// 表示 "as.of.instant = 2022-12-15 00:00:00"
spark.read.
  format("hudi").
  option("as.of.instant", "2022-12-15").
  load(basePath).
  show()
```

![image-20221215193721726](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221215193721726.png)

#### 更新数据

```scala
val updates = convertToStringList(dataGen.generateUpdates(10))
val df = spark.read.json(spark.sparkContext.parallelize(updates, 2))
df.write.format("hudi").
  options(getQuickstartWriteConfigs).
  option(PRECOMBINE_FIELD_OPT_KEY, "ts").
  option(RECORDKEY_FIELD_OPT_KEY, "uuid").
  option(PARTITIONPATH_FIELD_OPT_KEY, "partitionpath").
  option(TABLE_NAME, tableName).
  mode(Append).
  save(basePath)
```

保存模式现在是Append。通常，除非是第一次创建表，否则请始终使用追加模式。现在再次查询数据将显示更新的行程数据。每个写操作都会生成一个用时间戳表示的新提交。

![image-20221215195123418](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221215195123418.png)

#### 增量查询

增量查询的方式可以获取从给定提交时间戳以来更改的数据流。需要指定增量查询的beginTime，选择性指定endTime。如果我们希望在给定提交之后进行所有更改，则不需要指定endTime。

```scala
//重新加载数据
spark.
  read.
  format("hudi").
  load(basePath).
  createOrReplaceTempView("hudi_trips_snapshot")
//获取指定beginTime
val commits = spark.sql("select distinct(_hoodie_commit_time) as commitTime from  hudi_trips_snapshot order by commitTime").map(k => k.getString(0)).take(50)
val beginTime = commits(commits.length - 2) 
  
// 创建增量查询的表
val tripsIncrementalDF = spark.read.format("hudi").
  option(QUERY_TYPE_OPT_KEY, QUERY_TYPE_INCREMENTAL_OPT_VAL).
  option(BEGIN_INSTANTTIME_OPT_KEY, beginTime).
  load(basePath)
tripsIncrementalDF.createOrReplaceTempView("hudi_trips_incremental")
spark.sql("select `_hoodie_commit_time`, fare, begin_lon, begin_lat, ts from  hudi_trips_incremental where fare > 20.0").show()
```

![image-20221215200235437](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221215200235437.png)

#### 指定时间点查询

查询特定时间点的数据，可以将endTime指向特定时间，beginTime指向000（表示最早提交时间）

```scala
val beginTime = "000" 
val endTime = commits(commits.length - 2) 

val tripsPointInTimeDF = spark.read.format("hudi").
  option(QUERY_TYPE_OPT_KEY, QUERY_TYPE_INCREMENTAL_OPT_VAL).
  option(BEGIN_INSTANTTIME_OPT_KEY, beginTime).
  option(END_INSTANTTIME_OPT_KEY, endTime).
  load(basePath)
tripsPointInTimeDF.createOrReplaceTempView("hudi_trips_point_in_time")

spark.sql("select `_hoodie_commit_time`, fare, begin_lon, begin_lat, ts from hudi_trips_point_in_time where fare > 20.0").show()
```

![image-20221215200705599](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221215200705599.png)

#### 删除数据

根据传入的HoodieKeys来删除（uuid + partitionpath），只有append模式，才支持删除功能。

```scala
// 获取总数据 条数
spark.sql("select uuid, partitionpath from hudi_trips_snapshot").count()

// 选择两条数据删除
val ds = spark.sql("select uuid, partitionpath from hudi_trips_snapshot").limit(2)
//构建DataFrame
val deletes = dataGen.generateDeletes(ds.collectAsList())
val df = spark.read.json(spark.sparkContext.parallelize(deletes, 2))

//执行删除
df.write.format("hudi").
  options(getQuickstartWriteConfigs).
  option(OPERATION_OPT_KEY,"delete").
  option(PRECOMBINE_FIELD_OPT_KEY, "ts").
  option(RECORDKEY_FIELD_OPT_KEY, "uuid").
  option(PARTITIONPATH_FIELD_OPT_KEY, "partitionpath").
  option(TABLE_NAME, tableName).
  mode(Append).
  save(basePath)
//验证
val roAfterDeleteViewDF = spark.
  read.
  format("hudi").
  load(basePath)

roAfterDeleteViewDF.registerTempTable("hudi_trips_snapshot")

// 返回的总行数应该比原来少2行
spark.sql("select uuid, partitionpath from hudi_trips_snapshot").count()
```

![image-20221215203258783](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221215203258783.png)

####  覆盖数据

类似于hive的 “insert overwrite”

  操作，以忽略现有数据，只用提供的新数据创建一个提交。Insert overwrite操作可能比批量ETL作业的upsert更快，批量ETL作业是每一批次都要重新计算整个目标分区（包括索引、预组合和其他重分区步骤）。

````shell
spark.
  read.format("hudi").
  load(basePath).
  select("uuid","partitionpath").
  sort("partitionpath","uuid").
  show(100, false)
  
val inserts = convertToStringList(dataGen.generateInserts(10))
val df = spark.
  read.json(spark.sparkContext.parallelize(inserts, 2)).
  filter("partitionpath = 'americas/united_states/san_francisco'")
  
df.write.format("hudi").
  options(getQuickstartWriteConfigs).
  option(OPERATION.key(),"insert_overwrite").
  option(PRECOMBINE_FIELD.key(), "ts").
  option(RECORDKEY_FIELD.key(), "uuid").
  option(PARTITIONPATH_FIELD.key(), "partitionpath").
  option(TBL_NAME.key(), tableName).
  mode(Append).
  save(basePath)  
 
 spark.
  read.format("hudi").
  load(basePath).
  select("uuid","partitionpath").
  sort("partitionpath","uuid").
  show(100, false)
````

![image-20221215203717378](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221215203717378.png)

### Spark SQL

#### 启动Hive的Metastore

```shell
nohup hive --service metastore & 
```

#### 启动spark-sql

```shell
spark-sql \
  --conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer' \
  --conf 'spark.sql.catalog.spark_catalog=org.apache.spark.sql.hudi.catalog.HoodieCatalog' \
  --conf 'spark.sql.extensions=org.apache.spark.sql.hudi.HoodieSparkSessionExtension' \
```



| 参数名          | 默认值 | 说明                                                         |
| --------------- | ------ | ------------------------------------------------------------ |
| primaryKey      | uuid   | 表的主键名，多个字段用逗号分隔。同 hoodie.datasource.write.recordkey.field |
| preCombineField |        | 表的预合并字段。同 hoodie.datasource.write.precombine.field  |
| type            | cow    | 创建的表类型：  type = 'cow'  type = 'mor'同hoodie.datasource.write.table.type |

创建一个cow表，默认primaryKey 'uuid'，不提供preCombineField

```sql
create table hudi_cow_nonpcf_tbl (
  uuid int,
  name string,
  price double
) using hudi;
```

创建一个mor非分区表

```sql
create table hudi_mor_tbl (
  id int,
  name string,
  price double,
  ts bigint
) using hudi
tblproperties (
  type = 'mor',
  primaryKey = 'id',
  preCombineField = 'ts'
);
```

创建分区表

```sql
create table hudi_cow_pt_tbl (
  id bigint,
  name string,
  ts bigint,
  dt string,
  hh string
) using hudi
tblproperties (
  type = 'cow',
  primaryKey = 'id',
  preCombineField = 'ts'
 )
partitioned by (dt, hh)
location '/tmp/hudi/hudi_cow_pt_tbl';
```

#### 插入数据

##### 非分区

```sql
insert into hudi_spark.hudi_cow_nonpcf_tbl select 1, 'A1', 1;
insert into hudi_spark.hudi_mor_tbl select 1, 'A1', 10, 1000;
```

##### 分区表

```sql
-- 动态分区
insert into hudi_spark.hudi_cow_pt_tbl partition (dt, hh)
select 1 as id, 'A1' as name, 1000 as ts, '2023-01-03' as dt, '10' as hh;
-- 静态分区
insert into hudi_spark.hudi_cow_pt_tbl partition(dt = '2023-01-03', hh='22') select 2, 'A2', 1000;
```

使用`bulk_insert`插入数据

```sql
select id, name, price, ts from hudi_spark.hudi_mor_tbl;
```

![image-20230104205211904](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20230104205211904.png)

```sql
-- 向指定preCombineKey的表插入数据，则写操作为upsert
insert into hudi_spark.hudi_mor_tbl select 1, 'A1_1', 20, 1001;
```

![image-20230104205654585](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20230104205654585.png)

```sql
-- 向指定preCombineKey的表插入数据，指定写操作为bulk_insert 
set hoodie.sql.bulk.insert.enable=true;
set hoodie.sql.insert.mode=non-strict;

insert into hudi_spark.hudi_mor_tbl select 1, 'A1_2', 20, 1002;
select id, name, price, ts from hudi_spark.hudi_mor_tbl;
```

![image-20230104205809521](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20230104205809521.png)

###  查询

Hudi从`0.9.0`支持时间旅行查询。Spark SQL 要求Spark版本 3.2及以上

```sql
set hoodie.sql.bulk.insert.enable=false;

create table hudi_cow_pt_tbl1 (
  id bigint,
  name string,
  ts bigint,
  dt string,
  hh string
) using hudi
tblproperties (
  type = 'cow',
  primaryKey = 'id',
  preCombineField = 'ts'
 )
partitioned by (dt, hh)
location '/tmp/hudi/hudi_cow_pt_tbl1';


-- 插入一条id为1的数据
insert into hudi_cow_pt_tbl1 select 1, 'A0', 1000, '2023-01-04', '10';
select * from hudi_cow_pt_tbl1;
-- 修改id为1的数据
insert into hudi_cow_pt_tbl1 select 1, 'A1', 1001, '2023-01-04', '10';
select * from hudi_cow_pt_tbl1;
-- 基于第一次提交时间进行时间旅行
select * from hudi_cow_pt_tbl1 timestamp as of '20230104214452941' where id = 1;
-- 其他时间格式的时间旅行写法
select * from hudi_cow_pt_tbl1 timestamp as of '2023-01-04 21:44:52.941' where id = 1;
-- 在2023-01-05 00：00: 00之前的数据
select * from hudi_cow_pt_tbl1 timestamp as of '2023-01-05' where id = 1;
```

![image-20230104214755760](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20230104214755760.png)

### 更新数据

```sql
-- 更新语法
UPDATE tableIdentifier SET column = EXPRESSION(,column = EXPRESSION) [ WHERE boolExpression]
```



```sql
update hudi_cow_pt_tbl1 set name = 'A1_RENAME', ts = 1001 where id = 1;
```

![image-20230104221439466](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20230104221439466.png)

```sql
-- update using non-PK field
update hudi_cow_pt_tbl1 set ts = 1111 where name = 'A1_RENAME';
```

![image-20230104221905530](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20230104221905530.png)

```sql
update hudi_mor_tbl set price = price / 2, name = 'A1_UPDATE_NAME' where id = 1;
```

![image-20230104222139772](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20230104222139772.png)

### MergeInto

```sql
MERGE INTO tableIdentifier AS target_alias
USING (sub_query | tableIdentifier) AS source_alias
ON <merge_condition>
[ WHEN MATCHED [ AND <condition> ] THEN <matched_action> ]
[ WHEN MATCHED [ AND <condition> ] THEN <matched_action> ]
[ WHEN NOT MATCHED [ AND <condition> ]  THEN <not_matched_action> ]

<merge_condition> =A equal bool condition 
<matched_action>  =
  DELETE  |
  UPDATE SET *  |
  UPDATE SET column1 = expression1 [, column2 = expression2 ...]
<not_matched_action>  =
  INSERT *  |
  INSERT (column1 [, column2 ...]) VALUES (value1 [, value2 ...])
```

```sql
create table merge_source (
  id int,
  name string,
  price double,
  ts bigint) 
  using hudi
tblproperties (
  primaryKey = 'id', 
  preCombineField = 'ts'
);
insert into merge_source values (1, "OLD_A1_NAME", 22.22, 2900), (2, "NEW_A2_NAME", 22.22, 2000), (3, "NEW_A3_NAME", 33.33, 2000);

select * from merge_source;
insert into hudi_mor_tbl select 1, 'A1_2', 20, 1002;
select * from hudi_mor_tbl;

merge into hudi_mor_tbl as target
using merge_source as source
on target.id = source.id
when matched then update set *
when not matched then insert *
;
```

![image-20230104224011097](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20230104224011097.png)

![image-20230104224057171](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20230104224057171.png)

匹配关联到的数据全部更新，未匹配关联到的数据全部插入

需要开启hiverserver2服务

```sql
create table merge_source2 (id int, name string, flag string, dt string, hh string) using parquet;
insert into merge_source2 values (1, "NEW_A1", 'update', '2023-01-04', '10'), (2, "NEW_A2", 'delete', '2023-01-04', '11'), (3, "NEW_A3", 'insert', '2023-01-04', '12');

merge into hudi_cow_pt_tbl1 as target
using (
  select id, name, '2000' as ts, flag, dt, hh from merge_source2
) source
on target.id = source.id
when matched and flag != 'delete' then
 update set id = source.id, name = source.name, ts = source.ts, dt = source.dt, hh = source.hh
when matched and flag = 'delete' then delete
when not matched then
 insert (id, name, ts, dt, hh) values(source.id, source.name, source.ts, source.dt, source.hh);
```

![image-20230104230138328](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20230104230138328.png)



https://x.99ton.men/app/Clash/ClashA.apk

