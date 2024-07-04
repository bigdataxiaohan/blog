---
title: Hive数据据类型 DDL DML
date: 2019-01-16 18:24:31
tags: Hive
categories: 大数据
---

## 基本数据类型

| Hive数据类型 | Java数据类型 | 长度                                                 | 例子                                 |
| ------------ | ------------ | ---------------------------------------------------- | ------------------------------------ |
| TINYINT      | byte         | 1byte有符号整数                                      | 20                                   |
| SMALINT      | short        | 2byte有符号整数                                      | 20                                   |
| INT          | int          | 4byte有符号整数                                      | 20                                   |
| BIGINT       | long         | 8byte有符号整数                                      | 20                                   |
| BOOLEAN      | boolean      | 布尔类型，true或者false                              | TRUE  FALSE                          |
| FLOAT        | float        | 单精度浮点数                                         | 3.14159                              |
| DOUBLE       | double       | 双精度浮点数                                         | 3.14159                              |
| STRING       | string       | 字符系列。可以指定字符集。可以使用单引号或者双引号。 | ‘now is the time’ “for all good men” |
| TIMESTAMP    |              | 时间类型                                             |                                      |
| BINARY       |              | 字节数组                                             |                                      |

对于Hive而言String类型相当于数据库的varchar类型，该类型是一个可变的字符串，不过它不能声明其中最多能存储多少个字符，理论上它可以存储2GB的字符数。

###  集合数据类型

| 数据类型 | 描述                                                         | 语法示例 |
| -------- | ------------------------------------------------------------ | -------- |
| STRUCT   | 和c语言中的struct类似，都可以通过“点”符号访问元素内容。例如，如果某个列的数据类型是STRUCT{first STRING, last STRING},那么第1个元素可以通过字段.first来引用。 | struct() |
| MAP      | MAP是一组键-值对元组集合，使用数组表示法可以访问数据。例如，如果某个列的数据类型是MAP，其中键->值对是’first’->’John’和’last’->’Doe’，那么可以通过字段名[‘last’]获取最后一个元素 | map()    |
| ARRAY    | 数组是一组具有相同类型和名称的变量的集合。这些变量称为数组的元素，每个数组元素都有一个编号，编号从零开始。例如，数组值为[‘John’,   ‘Doe’]，那么第2个元素可以通过数组名[1]进行引用。 | Array()  |

Hive有三种复杂数据类型ARRAY、MAP 和 STRUCT。ARRAY和MAP与Java中的Array和Map类似，而STRUCT与C语言中的Struct类似，它封装了一个命名字段集合，复杂数据类型允许任意层次的嵌套。

### 类型转化

Hive的原子数据类型是可以进行隐式转换的，类似于Java的类型转换，例如某表达式使用INT类型，TINYINT会自动转换为INT类型，但是Hive不会进行反向转化，例如，某表达式使用TINYINT类型，INT不会自动转换为TINYINT类型，它会返回错误，除非使用CAST操作。

1．隐式类型转换规则如下

（1）任何整数类型都可以隐式地转换为一个范围更广的类型，如TINYINT可以转换成INT，INT可以转换成BIGINT。

（2）所有整数类型、FLOAT和STRING类型都可以隐式地转换成DOUBLE。

（3）TINYINT、SMALLINT、INT都可以转换为FLOAT。

（4）BOOLEAN类型不可以转换为任何其它的类型。

2．可以使用CAST操作显示进行数据类型转换

例如CAST('1' AS INT)将把字符串'1' 转换成整数1；如果强制类型转换失败，如执行CAST('X' AS INT)，表达式返回空值 NULL。

## DDL数据定义

###  创建数据库

```sql
hive> create database if not exists db_hive;      --最标准写法

hive> create database db_hive_HDFS location '/db_hive_hdfs.db';  --指定数据库在HDFS上存放的位置
```

![kS6mx1.png](https://s2.ax1x.com/2019/01/16/kS6mx1.png)

### 查询数据库

```sql
hive> show databases;
OK
db_hive
db_hive_hdfs
default
```

查看数据库详情

```sql
hive> desc database db_hive;
OK
db_hive         hdfs://datanode1:9000/user/hive/warehouse/db_hive.db    hadoop  USER
Time taken: 0.043 seconds, Fetched: 1 row(s)
hive> desc db_hive_hdfs;

hive> desc database db_hive_hdfs;
OK
db_hive_hdfs            hdfs://datanode1:9000/db_hive_hdfs.db   hadoop  USER
Time taken: 0.041 seconds, Fetched: 1 row(s)
```

切换当前数据库

```shell
hive> use db_hive;
```

### 修改数据库

用户可以使用ALTER DATABASE命令为某个数据库的DBPROPERTIES设置键-值对属性值，来描述这个数据库的属性信息。数据库的其他元数据信息都是不可更改的，包括数据库名和数据库所在的目录位置。**修改当前正在使用的数据库，要先退出使用**

```sql
hive> use default;
OK
Time taken: 0.038 seconds

hive>  alter database db_hive set dbproperties('createtime'='20190116') ;
OK
Time taken: 0.105 seconds

hive> desc database extended db_hive;
OK
db_hive         hdfs://datanode1:9000/user/hive/warehouse/db_hive.db    hadoop  USER    {createtime=20190116}
Time taken: 0.064 seconds, Fetched: 1 row(s)
```

### 删除数据库

```sql
hive> drop database db_hive_hdfs;					   --删除空数据库
hive> drop database if exists db_hive_hdfs;				--这么些删除不存在的不会报错
hive> drop database db_hive cascade;				    --如果数据库非空可以强制删除
```

### 创建表

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
[(col_name data_type [COMMENT col_comment], ...)] 
[COMMENT table_comment] 
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
[CLUSTERED BY (col_name, col_name, ...) 
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 
[ROW FORMAT row_format] 
[STORED AS file_format] 
[LOCATION hdfs_path]
```

| 参数                                                         | 功能                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <font color=#0099ff size=2 face="黑体">CREATE  TABLE</font>  | <font color=#0099ff size=2 face="黑体">创建一个指定名字的表。如果表存在，抛出异常，用 IF NOT EXISTS忽略异常。</font> |
| <font color=#0099ff size=2 face="黑体">EXTERNA</font>        | <font color=#0099ff size=2 face="黑体">用户创建一个外部表，在建表的同时指定一个指向实际数据的路径（LOCATION），Hive创建内部表时，会将数据移动到数据仓库指向的路径；若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。</font> |
| <font color=#0099ff size=2 face="黑体">PARTITIONED BY</font> | <font color=#0099ff size=2 face="黑体">创建分区表</font>     |
| <font color=#0099ff size=2 face="黑体">COMMENT</font>        | <font color=#0099ff size=2 face="黑体">为表和列添加注释。</font> |
| <font color=#0099ff size=2 face="黑体">CLUSTERED BY</font>   | <font color=#0099ff size=2 face="黑体">创建分桶表</font>     |
| <font color=#0099ff size=2 face="黑体">SORTED BY</font>      | <font color=#0099ff size=2 face="黑体">不常用</font>         |
| <font color=#0099ff size=2 face="黑体">ROW FORMAT</font>     | <font color=#0099ff size=2 face="黑体"> 用户在建表的时候可以自定义SerDe或者使用自带的SerDe。如果没有指定ROW FORMAT 或者ROW FORMAT<br/>DELIMITED，将会使用自带的SerDe。在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的SerDe，Hive通过SerDe确定表的具体的列的数据。 </font> |
| <font color=#0099ff size=2 face="黑体">ROW FORMAT</font>     | <font color=#0099ff size=2 face="黑体">ROW FORMAT</font>     |
| <font color=#0099ff size=2 face="黑体">STORED AS</font>      | <font color=#0099ff size=2 face="黑体">指定存储文件类型常用的存储文件类型：SEQUENCEFILE（二进制序列文件）、TEXTFILE（文本）、RCFILE（列式存储格式文件）</font> |
| <font color=#0099ff size=2 face="黑体">LOCATION</font>       | <font color=#0099ff size=2 face="黑体">指定表在HDFS上的存储位置。</font> |
| <font color=#0099ff size=2 face="黑体">LIKE </font>          | <font color=#0099ff size=2 face="黑体">LIKE允许用户复制现有的表结构，但是不复制数据。</font> |

### 管理表

默认创建的表都是管理表（内部表）。这种表，Hive会（或多或少地）控制着数据的生命周期。Hive默认情况下会将这些表的数据存储在由配置项hive.metastore.warehouse.dir(例如，/user/hive/warehouse)所定义的目录的子目录下。当我们删除一个管理表时，Hive也会删除这个表中数据。管理表不适合和其他工具共享数据。

创建普通表

```sql
create table if not exists student2(
id int, name string
)
row format delimited fields terminated by '\t'
stored as textfile
location '/user/hive/warehouse/student2';
```

根据查询结果创建表（查询的结果会添加到新创建的表中）会进行mapreduce作业

```sql
create table if not exists student3 as select id, name from student;
```

根据已经存在的表结构创建表

```sql
create table if not exists student4 like student;
```

查询表的详细类型

```
desc formatted student2;
```

### 外部表

外部表Hive并非认为其完全拥有这份数据。删除该表并不会删除掉这份数据，表的元数据信息会被删除掉。

### 应用场景

每天将收集到的网站日志定期流入HDFS文本文件。在外部表（原始日志表）的基础上做大量的统计分析，用到的中间表、结果表使用内部表存储，数据通过SELECT+INSERT进入内部表。

### 案例实操

数据准备

```shell
[hadoop@datanode1 datas]$ vim dept.txt

10      ACCOUNTING      1700
20      RESEARCH        1800
30      SALES   1900
40      OPERATIONS      1700
--------------------------------------------------------------------------------------------------
[hadoop@datanode1 datas]$ vim emp.txt
7369    SMITH   CLERK   7902    1980-12-17      800.00          20
7499    ALLEN   SALESMAN        7698    1981-2-20       1600.00 300.00  30
7521    WARD    SALESMAN        7698    1981-2-22       1250.00 500.00  30
7566    JONES   MANAGER 7839    1981-4-2        2975.00         20
7654    MARTIN  SALESMAN        7698    1981-9-28       1250.00 1400.00 30
7698    BLAKE   MANAGER 7839    1981-5-1        2850.00         30
7782    CLARK   MANAGER 7839    1981-6-9        2450.00         10
7788    SCOTT   ANALYST 7566    1987-4-19       3000.00         20
7839    KING    PRESIDENT               1981-11-17      5000.00         10
7844    TURNER  SALESMAN        7698    1981-9-8        1500.00 0.00    30
7876    ADAMS   CLERK   7788    1987-5-23       1100.00         20
7900    JAMES   CLERK   7698    1981-12-3       950.00          30
7902    FORD    ANALYST 7566    1981-12-3       3000.00         20
7934    MILLER  CLERK   7782    1982-1-23       1300.00         10
```

创建表 

````sql
--部门表
create external table if not exists default.dept(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t';

--员工表
create external table if not exists default.emp(
empno int,
ename string,
job string,
mgr int,
hiredate string, 
sal double, 
comm double,
deptno int)
row format delimited fields terminated by '\t';

--查询表
hive> show tables;
OK
dept
student
student2
student3
student4
Time taken: 0.04 seconds, Fetched: 5 row(s)

--加载数据
hive> load data local inpath '/opt/module/datas/dept.txt' into table default.dept;
Loading data to table default.dept
Table default.dept stats: [numFiles=1, totalSize=69]
OK
Time taken: 0.569 seconds

hive> load data local inpath '/opt/module/datas/emp.txt' into table default.emp;
Loading data to table default.emp
Table default.emp stats: [numFiles=1, totalSize=657]
OK
Time taken: 0.604 seconds

--查询结果
hive> select * from emp;
hive> select * from dept;

--查看表格信息
desc formatted dept;
````

### 互相转换

注意：**只能用单引号，严格区分大小写，如果不是完全符合，那么只会添加kv 而不生效**

```sql
 --查询表类型
 desc formatted student2;
 Table Type:             MANAGED_TABLE
--修改内部表student2为外部表
alter table student2 set tblproperties('EXTERNAL'='TRUE');
--查询表的类型
desc formatted student2;
Table Type:             EXTERNAL_TABLE
--修改外部表student2为内部表
alter table student2 set tblproperties('EXTERNAL'='FALSE');
--查询表的类型
desc formatted student2;
Table Type:             MANAGED_TABLE
```

注意：**注意：('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写！**

## 分区表

分区表实际上就是对应一个HDFS文件系统上的独立的文件夹，该文件夹下是该分区所有的数据文件。**Hive中的分区就是分目录**，把一个大的数据集根据业务需要分割成小的数据集。在查询时通过WHERE子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。

### 创建分区表

```sql
create table dept_partition(
deptno int, dname string, loc string
)
partitioned by (month string)
row format delimited fields terminated by '\t';
```

```sql
load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201901');
load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201902');
load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201903');
```

![kShcAP.png](https://s2.ax1x.com/2019/01/16/kShcAP.png)

### 查询分区表

```sql
--单分区查询
select * from dept_partition where month='201901';
-- 多分区联合查询  union（排序）    or   in 三种方式
select * from dept_partition where month='201901'
              union
              select * from dept_partition where month='201902'
              union
              select * from dept_partition where month='201903';
 
 -------------------------------------------------------------------------------------------------
Hadoop job information for Stage-2: number of mappers: 2; number of reducers: 1
2019-01-16 21:02:28,578 Stage-2 map = 0%,  reduce = 0%
2019-01-16 21:02:48,169 Stage-2 map = 50%,  reduce = 0%, Cumulative CPU 2.29 sec
2019-01-16 21:02:49,219 Stage-2 map = 100%,  reduce = 0%, Cumulative CPU 5.17 sec
2019-01-16 21:02:58,782 Stage-2 map = 100%,  reduce = 100%, Cumulative CPU 7.58 sec
MapReduce Total cumulative CPU time: 7 seconds 580 msec
Ended Job = job_1547632574371_0003
MapReduce Jobs Launched:
Stage-Stage-1: Map: 2  Reduce: 1   Cumulative CPU: 7.82 sec   HDFS Read: 15318 HDFS Write: 410 SUCCESS
Stage-Stage-2: Map: 2  Reduce: 1   Cumulative CPU: 7.58 sec   HDFS Read: 15080 HDFS Write: 291 SUCCESS
Total MapReduce CPU Time Spent: 15 seconds 400 msec
OK
10      ACCOUNTING      1700    201901
10      ACCOUNTING      1700    201902
10      ACCOUNTING      1700    201903
20      RESEARCH        1800    201901
20      RESEARCH        1800    201902
20      RESEARCH        1800    201903
30      SALES   1900    201901
30      SALES   1900    201902
30      SALES   1900    201903
40      OPERATIONS      1700    201901
40      OPERATIONS      1700    201902
40      OPERATIONS      1700    201903
Time taken: 100.192 seconds, Fetched: 12 row(s)
```

### 增加分区

```sql
--添加单个分区
alter table dept_partition add partition(month='201904') ;
--增加分区  用空格分开
alter table dept_partition drop partition (month='201802')  partition (month='201803');
```

### 删除分区

```sql
--删除单个分区
 alter table dept_partition drop partition (month='201801');
 
--同时删除多个分区  用逗号分开
alter table dept_partition drop partition (month='201802'), partition (month='201803');
```

### 查看分区

```sql
-- show partitions dept_partition;
hive> show partitions dept_partition;
OK
month=201801
month=201901
month=201902
month=201903
month=201904
Time taken: 0.123 seconds, Fetched: 5 row(s)

--查看分区表结构
hive> desc formatted dept_partition;
OK
# col_name              data_type               comment

deptno                  int
dname                   string
loc                     string

# Partition Information
# col_name              data_type               comment

month                   string
```

### 分区表注意事项

#### 二级分区表

```sql
--创建二级分区表
create table dept_partition2(deptno int, dname string, loc string )
partitioned by (month string, day string)
row format delimited fields terminated by '\t';

--加载数据到二级分区表
load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition2 partition(month='201812', day='31');

--查询分区数据
select * from dept_partition2 where month='201812' and day='31';
```

#### 数据关联

1.上传数据后修复

```sql
dfs -mkdir -p  /user/hive/warehouse/dept_partition2/month=201812/day=26;
dfs -put /opt/module/datas/dept.txt  /user/hive/warehouse/dept_partition2/month=201812/day=26;

--修复命令
hive> msck repair table dept_partition2;
OK
Partitions not in metastore:    dept_partition2:month=201812/day=26
Repair: Added partition to metastore dept_partition2:month=201812/day=26
Time taken: 0.294 seconds, Fetched: 2 row(s)

--查询  如果没有执行修复命令  刚开始是查询不到的
hive> select * from dept_partition2 where month='201812' and day='26';
```

2.上传数据后修复

```sql
--上传数据
dfs -mkdir -p  /user/hive/warehouse/dept_partition2/month=201809/day=11;
dfs -put /opt/module/datas/dept.txt  /user/hive/warehouse/dept_partition2/month=201809/day=11;

--执行添加分区
alter table dept_partition2 add partition(month='201809',day='11');

--查询数据
 select * from dept_partition2 where month='201809' and day='11';
```

3.上传数据后load数据到分区

```sql
--创建目录
dfs -mkdir -p  /user/hive/warehouse/dept_partition2/month=201810/day=25;

--上传数据
load data local inpath '/opt/module/datas/dept.txt' into table  dept_partition2 partition(month='201810',day='25');

--查询数据
select * from dept_partition2 where month='201810' and day='25';
```

#### 修改表

```sql
--语法
ALTER TABLE table_name RENAME TO new_table_name

--实例
hive> alter table dept_partition2 rename to dept_partition_rename;
```

#### 增加/修改/替换列信息

```sql
--更新列
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name]

--增加和替换列
ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...) 
```

**注：ADD是代表新增一字段，字段位置在所有列后面(partition列前)，REPLACE则是表示替换表中所有字段。**

实际案例

```sql
--查询表结构
desc dept_partition;

--添加列
alter table dept_partition add columns(deptdesc string);

--查询表结构
desc dept_partition;

--更新列
alter table dept_partition change column deptdesc desc int;

--查询表结构
desc dept_partition;

--替换列
alter table dept_partition replace columns(deptno string, dname,string, loc string);

--替换列
 desc dept_partition;
```

#### 删除表

```sql
--语法
drop table 表名;

--实例
drop table dept_partition;
```

### DML数据操作

#### 向表中装载数据（Load）

```sql
load data [local] inpath '/opt/module/datas/student.txt' [overwrite] into table student [partition (partcol1=val1,…)];
```

| 参数           | 意义                                                         |
| -------------- | ------------------------------------------------------------ |
| load data      | 表示加载数据                                                 |
| local          | 从本地加载数据到hive表（复制）；否则从HDFS加载数据到hive表（移动） |
| inpath         | 表示加载数据的路径                                           |
| overwrite into | 表示覆盖表中已有数据，否则表示追加                           |
| into table     | 表示加载到哪张表                                             |
| student        | 表示具体的表                                                 |
| partition      | 表示上传到指定分区                                           |

```sql
--创建表
create table student(id string, name string) row format delimited fields terminated by '\t';

--加载本地文件到hive
 load data local inpath '/opt/module/datas/student.txt' into table default.student;

********************************* 加载HDFS文件到hive *********************************
--上传文件到HDFS
hive> dfs -put /opt/module/datas/student.txt /user/hadoop/hive;

--加载HDFS上数据
load data inpath '/user/hadoop/hive/student.txt' into table default.student;

********************************* 加载数据覆盖表中已有的数据 *********************************

--加载数据覆盖表中已有的
load data inpath '/user/hadoop/hive/student.txt' overwrite into table default.student;
```

####  通过查询语句向表中插入数据（Insert）

```sql
 --创建一张分区表
 create table student(id int, name string) partitioned by (month string) row format delimited fields terminated by '\t';
 
 --基本插入数据
 insert into table  student partition(month='201901') values(1,'Hive');
 
 --基本模式插入（根据单张表查询结果）
insert overwrite table student partition(month='201812') select id, name from student where month='201901';

--多插入模式（根据多张表查询结果）
from student
              insert overwrite table student partition(month='201902')
              select id, name where month='201901'
              insert overwrite table student partition(month='201903')
              select id, name where month='201901';
```

#### 查询语句中创建表并加载数据（As Select）

```sql
--创建表，并指定在hdfs上的位置
create table if not exists student5(
              id int, name string
              )
              row format delimited fields terminated by '\t'
              location '/user/hive/warehouse/student5';

--上传数据到hdfs上
dfs -put /opt/module/datas/student.txt /user/hive/warehouse/student5;

--查询
 select * from student5;
 hive> select * from student5;
OK
1       hadoop
2       spark
3       flink
4       oozie
5       java
6       python
7       MachineLearning
8       Scala
9       Hive
Time taken: 0.082 seconds, Fetched: 9 row(s)
```



```sql
 --导出数据
[hadoop@datanode1 hive]$  bin/hive -e "EXPORT TABLE student    TO '/export/hive/student';"

--查看表
hive> show tables;
OK
dept
dept_partition
dept_partition_rename
emp
student
student3
student4
student5
Time taken: 0.019 seconds, Fetched: 8 row(s)

--导入数据
import table student_import partition(month='201901') from '/export/hive/student';

--查看表
hive> show tables;
OK
dept
dept_partition
dept_partition_rename
emp
student
student3
student4
student5
student_import
Time taken: 0.019 seconds, Fetched: 9 row(s)

--查询数据
hive> select * from student2;
```



####  Insert导出

```sql
--将查询的结果导出到本地
insert overwrite local directory '/opt/module/datas/export/student'  select * from student;

--将查询的结果格式化导出到本地
insert overwrite local directory '/opt/module/datas/export/student1'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'    select * from student;

--将查询的结果导出到HDFS上(没有local)
insert overwrite directory '/export/hive/student2' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
select * from student;
```

#### 命令导出

```sql
-- Hadoop命令导出到本地

dfs -get /user/hive/warehouse/student/month=201901/000000_0  /opt/module/datas/export/student3.txt;

-- Hive Shell 命令导出
--基本语法：（hive -f/-e 执行语句或者脚本 > file）
[hadoop@datanode1 hive]$ bin/hive -e 'select * from default.student;' > /opt/module/datas/export/student4.txt;

--Export导出到HDFS上
hive> export table default.student to '/export/hive/student';
```

#### Sqoop导出

[Sqoop参考](http://hphblog.cn/2019/01/08/Sqoop/)





 









