---
title: Hive查询
date: 2019-01-17 11:11:26
tags: Hive
categories: 大数据
---

## 查询语法

```sql
[WITH CommonTableExpression (, CommonTableExpression)*]    (Note: Only available
 starting with Hive 0.13.0)
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
  FROM table_reference
  [WHERE where_condition]
  [GROUP BY col_list]
  [ORDER BY col_list]
  [CLUSTER BY col_list
    | [DISTRIBUTE BY col_list] [SORT BY col_list]
  ]
 [LIMIT number]
```

[参考文档](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select)

## 基本查询

```sql
select  …  from
```

### 全表查询

```
select * from emp;
```

### 条件查询

```
select empno, ename from emp;
```

注意:

1. SQL 语言大小写不敏感。 
2. SQL 可以写在一行或者多行
3. 关键字不能被缩写也不能分行
4. 各子句一般要分行写。
5. 使用缩进提高语句的可读性。

###  列别名

1. 重命名一个列
2. 便于计算
3. 紧跟列名，也可以在列名和别名之间加入关键字‘AS’ 

```sql
--询名称和部门
hive > select ename AS name, deptno dn from emp;
```

### 算术运算符

| 运算符 | 描述           |
| ------ | -------------- |
| A+B    | A和B   相加    |
| A-B    | A减去B         |
| A*B    | A和B   相乘    |
| A/B    | A除以B         |
| A%B    | A对B取余       |
| A&B    | A和B按位取与   |
| A\|B   | A和B按位取或   |
| A^B    | A和B按位取异或 |
| ~A     | A按位取反      |

```sql
--薪水加100后的信息
select ename,sal +100 from emp;
```

### 常用函数

```sql
--求总行数（count）
select count(*) cnt from emp;

--求工资的最大值（max）
select max(sal) max_sal from emp;

--求工资的最小值（min）
select min(sal) min_sal from emp;

--求工资的总和（sum）
select sum(sal) sum_sal from emp; 

--求工资的平均值（avg）
select sum(sal) sum_sal from emp; 

--Limit语句
select * from emp limit 5;
```

### Where语句

1．使用WHERE子句，将不满足条件的行过滤掉

2．WHERE子句紧随FROM子句

```sql
--查询出薪水大于1000的所有员工
select * from emp where sal >1000;
```

### 比较运算符

| 操作符                  | 支持的数据类型 | 描述                                                         |
| ----------------------- | -------------- | ------------------------------------------------------------ |
| A=B                     | 基本数据类型   | 如果A等于B则返回TRUE，反之返回FALSE                          |
| A<=>B                   | 基本数据类型   | 如果A和B都为NULL，则返回TRUE，其他的和等号（=）操作符的结果一致，如果任一为NULL则结果为NULL |
| A<>B, A!=B              | 基本数据类型   | A或者B为NULL则返回NULL；如果A不等于B，则返回TRUE，反之返回FALSE |
| A<B                     | 基本数据类型   | A或者B为NULL，则返回NULL；如果A小于B，则返回TRUE，反之返回FALSE |
| A<=B                    | 基本数据类型   | A或者B为NULL，则返回NULL；如果A小于等于B，则返回TRUE，反之返回FALSE |
| A>B                     | 基本数据类型   | A或者B为NULL，则返回NULL；如果A大于B，则返回TRUE，反之返回FALSE |
| A>=B                    | 基本数据类型   | A或者B为NULL，则返回NULL；如果A大于等于B，则返回TRUE，反之返回FALSE |
| A [NOT] BETWEEN B AND C | 基本数据类型   | 如果A，B或者C任一为NULL，则结果为NULL。如果A的值大于等于B而且小于或等于C，则结果为TRUE，反之为FALSE。如果使用NOT关键字则可达到相反的效果。 |
| A IS NULL               | 所有数据类型   | 如果A等于NULL，则返回TRUE，反之返回FALSE                     |
| A IS NOT NULL           | 所有数据类型   | 如果A不等于NULL，则返回TRUE，反之返回FALSE                   |
| IN(数值1, 数值2)        | 所有数据类型   | 使用 IN运算显示列表中的值                                    |
| A [NOT] LIKE B          | STRING 类型    | B是一个SQL下的简单正则表达式，如果A与其匹配的话，则返回TRUE；反之返回FALSE。B的表达式说明如下：‘x%’表示A必须以字母‘x’开头，‘%x’表示A必须以字母’x’结尾，而‘%x%’表示A包含有字母’x’,可以位于开头，结尾或者字符串中间。如果使用NOT关键字则可达到相反的效果。 |
| A RLIKE B, A REGEXP B   | STRING 类型    | B是一个正则表达式，如果A与其匹配，则返回TRUE；反之返回FALSE。匹配使用的是JDK中的正则表达式接口实现的，因为正则也依据其中的规则。例如，正则表达式必须和整个字符串A相匹配，而不是只需与其字符串匹配。 |

```sql
--查询出薪水等于5000的所有员工
select * from emp where sal =5000;

--查询工资在5000到10000的员工信息
 select * from emp where sal  between 500 and 10000;
 
 --查询comm为空的所有员工信
 select * from emp where comm is null;
 
 --查询工资是1500或5000的员工信息
select * from emp where sal in(1500,5000);
select * from emp where sal=1500 or sal= 5000;
```

### Like和RLike

1. 使用LIKE运算选择类似的值
2. 选择条件可以包含字符或数字:% 代表零个或多个字符(任意个字符)。_ 代表一个字符。
3. RLIKE子句是Hive中这个功能的一个扩展，其可以通过Java的正则表达式这个更强大的语言来指定匹配条件。

```sql
-- 查找以2开头薪水的员工信息
 select * from emp where sal LIKE '2%';
-- 查找第二个数值为2的薪水的员工信息
select * from emp where sal LIKE '_2%';
-- 查找薪水中含有2的员工信息
select * from emp where sal RLIKE '[2]';
```

### 逻辑运算符

| 操作符 | 含义   |
| ------ | ------ |
| AND    | 逻辑并 |
| OR     | 逻辑或 |
| NOT    | 逻辑否 |

```sql
--求每个部门的平均工资
select deptno, avg(sal) from emp group by deptno;
--求每个部门的平均薪水大于2000的部门
select deptno, avg(sal) avg_sal from emp group by deptno having  avg_sal > 2000;
```

### Join语句

#### 等值Join

注意:Hive支持通常的SQL JOIN语句，但 **只支持等值连接，不支持非等值连接**。

```sql
--根据员工表和部门表中的部门编号相等，查询员工编号、员工名称和部门名称；
 select e.empno, e.ename, d.deptno, d.dname from emp e join dept d on e.deptno = d.deptno;
--合并员工表和部门表
 select e.empno, e.ename, d.deptno from emp e join dept d on e.deptno = d.deptno;
```

 表的别名:（1）使用别名可以简化查询。（2）使用表名前缀可以提高执行效率。

#### 左外连接

左外连接：JOIN操作符左边表中符合WHERE子句的所有记录将会被返回。

```sql
select e.empno, e.ename, d.deptno from emp e left join dept d on e.deptno = d.deptno;
```

#### 右外连接

右外连接：JOIN操作符右边表中符合WHERE子句的所有记录将会被返回。

```sql
select e.empno, e.ename, d.deptno from emp e right join dept d on e.deptno = d.deptno;
```

#### 满外连接

满外连接：将会返回所有表中符合WHERE语句条件的所有记录。如果任一表的指定字段没有符合条件的值的话，那么就使用NULL值替代。

```sql
select e.empno, e.ename, d.deptno from emp e full join dept d on e.deptno = d.deptno;
```

#### 多表连接

数据准备

```shell
[hadoop@datanode1 datas]$ vim location.txt
1700    Beijing
1800    London
1900    Tokyo
```

创建位置表

```sql
create table if not exists default.location(
loc int,
loc_name string
)
row format delimited fields terminated by '\t';
```

导入数据

```sql
 load data local inpath '/opt/module/datas/location.txt' into table default.location;
```

多表连接查询

```sql
SELECT e.ename, d.deptno, l. loc_name
FROM   emp e 
JOIN   dept d
ON     d.deptno = e.deptno 
JOIN   location l
ON     d.loc = l.loc;
```

注意:大多数情况下，Hive会对每对JOIN连接对象启动一个MapReduce任务。本例中会首先启动一个MapReduce job对表e和表d进行连接操作，然后会再启动一个MapReduce job将第一个MapReduce job的输出和表l;进行连接操作。Hive总是按照从左到右的顺序执行的。因此不是Hive总是按照从左到右的顺序执行的。

####  笛卡尔积

笛卡尔集会在下面条件下产生:

1. 省略连接条件
2. 连接条件无效
3. 所有表中的所有行互相连接

```sql
--案例实操
 select empno, dname from emp, dept;
 
 --连接谓词中不支持or
select e.empno, e.ename, d.deptno from emp e join dept d on e.deptno= d.deptno or e.ename=d.ename; ## 错误的
```

### 排序

#### 全局排序

Order By：全局排序，一个Reducer

1．使用 ORDER BY 子句排序
ASC（ascend）: 升序（默认）
DESC（descend）: 降序

ORDER BY 子句在SELECT语句的结尾

```sql
--查询员工信息按工资升序排列
hive (default)> select * from emp order by sal;
--查询员工信息按工资降序排列
select * from emp order by sal desc;
```

#### 按照别名排序

```sql
--按照员工薪水的2倍排序
select ename, sal*2 twosal from emp order by twosal;
--按照部门和工资升序排序
select ename, deptno, sal from emp order by deptno, sal;
```

#### MR内部排序（Sort By）

Sort By：每个Reducer内部进行排序，对全局结果集来说不是排序。

```sql
--设置reduce个数
set mapreduce.job.reduces=3;

--查看设置reduce个数
set mapreduce.job.reduces;

--根据部门编号降序查看员工信息
select empno,ename,sal,deptno from emp sort by empno desc;

--按照部门编号降序排序
select empno,ename,sal,deptno from emp sort by deptno desc;
```

#### 分区排序 （Distribute By）

Distribute By：类似MR中partition，进行分区，结合sort by使用。

注意：Hive要求DISTRIBUTE BY语句要写在SORT BY语句之前。对于distribute by进行测试，一定要分配多reduce进行处理，否则无法看到distribute by的效果。

案例实操：

```sql
--需求：先按照部门编号分区，再按照员工编号降序排序。
 set mapreduce.job.reduces=3;
 select * from emp distribute by deptno sort by empno desc;
```

#### Cluster By

当distribute by和sorts by字段相同时，可以使用cluster by方式。

cluster by除了具有distribute by的功能外还兼具sort by的功能。但是排序只能是倒序排序，不能指定排序规则为ASC或者DESC。

```sql
select * from emp cluster by deptno;
select * from emp distribute by deptno sort by deptno;
```

### 桶表

**分区针对的是数据的存储路径；分桶针对的是数据文件。**

分区提供一个隔离数据和优化查询的便利方式。不过，并非所有的数据集都可形成合理的分区，特别是之前所提到过的要确定合适的划分大小这个疑虑。 分桶是将数据集分解成更容易管理的若干部分的另一个技术。

```sql
--创建分桶表
create table stu_buck(id int, name string)
clustered by(id) 
into 4 buckets
row format delimited fields terminated by '\t';

Num Buckets:            4
--加载数据
 load data local inpath '/opt/module/datas/student.txt' into table  stu_buck;
```

查看创建的分桶表

![kpDZqO.png](https://s2.ax1x.com/2019/01/17/kpDZqO.png)

```sql
--创建分桶表时，数据通过子查询的方式导入
create table stu(id int, name string)
row format delimited fields terminated by '\t';

--向普通的stu表中导入数据
load data local inpath '/opt/module/datas/student.txt' into table stu;

--清空stu_buck表中数据
truncate table stu_buck;

--导入数据到分桶表，通过子查询的方式
insert into table stu_buck select id, name from stu;
```

![kpD5S1.png](https://s2.ax1x.com/2019/01/17/kpD5S1.png)

```sql
--为什么没有分桶呢 这里需要我们开启一个属性
 insert into table stu_buck select id, name from stu;
```

![kpDxSI.png](https://s2.ax1x.com/2019/01/17/kpDxSI.png)

```sql
--查询分桶的数据
hive> select * from stu_buck;
OK
1016    ss16
1012    ss12
1008    ss8
1004    ss4
1009    ss9
1005    ss5
1001    ss1
1013    ss13
1010    ss10
1002    ss2
1006    ss6
1014    ss14
1003    ss3
1011    ss11
1007    ss7
1015    ss15
Time taken: 0.091 seconds, Fetched: 16 row(s)
```

![kpDxSI.png](https://s2.ax1x.com/2019/01/17/kpDxSI.png)

![kps4r6.png](https://s2.ax1x.com/2019/01/17/kps4r6.png)

#### 分桶抽样查询

对于非常大的数据集，有时用户需要使用的是一个具有代表性的查询结果而不是全部结果。Hive可以通过对表进行抽样来满足这个需求。

查询表stu_buck中的数据     select * from 表名 tablesample(bucket x out of y on id);   

y必须是table总bucket数的倍数或者因子。hive根据y的大小，决定抽样的比例。例如，table总共分了4份，当y=2时，抽取(4/2=)2个bucket的数据，当y=8时，抽取(4/8=)1/2个bucket的数据。

x表示从哪个bucket开始抽取，如果需要取多个分区，以后的分区号为当前分区号加上y。例如，table总bucket数为4，tablesample(bucket 1 out of 2)，表示总共抽取（4/2=）2个bucket的数据，抽取第1(x)个和第4(x+y)个bucket的数据。

```sql
--抽取第一个分区的数据
hive> select * from stu_buck  tablesample(bucket 1 out of 4 on id);
OK
1016    ss16
1012    ss12
1008    ss8
1004    ss4
--查询第一个分区一半的数据
hive> select * from stu_buck  tablesample(bucket 1 out of 8 on id);
OK
1016    ss16
1008    ss8

--x的值必须小于等于y的值，否则
hive> select * from stu_buck  tablesample(bucket 6 out of 3 on id);
FAILED: SemanticException [Error 10061]: Numerator should not be bigger than denominator in sample clause for table stu_buck
```

###  其他常用查询函数

####  空字段赋值

NVL：给值为NULL的数据赋值，它的格式是NVL( string1,replace_with)。它的功能是如果string1为NULL，则NVL函数返回replace_with的值，否则返回string1的值，如果两个参数都为NULL ，则返回NULL。

```sql
--NULL用'无'代替
hive> select ename, deptno, sal, nvl(comm,'无') from emp;
OK
SMITH   20      800.0   无
ALLEN   30      1600.0  300.0
WARD    30      1250.0  500.0
JONES   20      2975.0  无
MARTIN  30      1250.0  1400.0
BLAKE   30      2850.0  无
CLARK   10      2450.0  无
SCOTT   20      3000.0  无
KING    10      5000.0  无
TURNER  30      1500.0  0.0
ADAMS   20      1100.0  无
JAMES   30      950.0   无
FORD    20      3000.0  无
MILLER  10      1300.0  无
```

#### CASE WHEN

```sql
--数据准备
[hadoop@datanode1 datas]$ vim emp_sex.txt
悟空    A       男
八戒    A       男
沙和尚  B       男
唐僧    A       女
白龙马  B       女
白骨精  B       女

--建表加载数据
create table person_info(
name string, 
constellation string, 
blood_type string) 
row format delimited fields terminated by "\t";

--查询
select 
 t1.c_b,
 CONCAT_WS("|",COLLECT_SET(t1.name))
from (
  select 
   CONCAT_WS(",",constellation,blood_type) c_b,
   name 
  from person_info) t1
group by 
t1.c_b;
--结果

dept_id male    female
A       2       1
B       1       2
```

#### 行转列

CONCAT(string A/col, string B/col…)：返回输入字符串连接后的结果，支持任意个输入字符串;

CONCAT_WS(separator, str1, str2,...)：它是一个特殊形式的 CONCAT()。第一个参数剩余参数间的分隔符。分隔符可以是与剩余参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间;

COLLECT_SET(col)：函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生array类型字段。

```sql
##需求:把星座和血型一样的人归类到一起
[hadoop@datanode1 datas]$ vim person_info.txt
孙悟空  白羊座  A
唐僧    射手座  A
沙和尚  白羊座  B
猪八戒  白羊座  A
白龙马  射手座  A

--建表导入数据
create table person_info(
name string, 
constellation string, 
blood_type string) 
row format delimited fields terminated by "\t";
load data local inpath "/opt/module/datas/ constellation.txt" into table person_info;

--查询
select t1.base,
    concat_ws('|', collect_set(t1.name)) name
from
    (select
        name,
        concat(constellation, ",", blood_type) base
    from
        person_info) t1
group by
    t1.base;
--结果
射手座,A        唐僧|白龙马
白羊座,A        孙悟空|猪八戒
白羊座,B        沙和尚
```

#### 列转行

EXPLODE(col)：将hive一列中复杂的array或者map结构拆分成多行。

LATERAL VIEW用法：LATERAL VIEW udtf(expression) tableAlias AS columnAlias

解释：用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。

```sql
--准备数据
[hadoop@datanode1 datas]$ vim movie.txt
《疑犯追踪》    悬疑,动作,科幻,剧情
《Lie to me》   悬疑,警匪,动作,心理,剧情
《战狼2》       战争,动作,灾难

--创建表加载数据
create table movie_info(
    movie string, 
    category array<string>) 
row format delimited fields terminated by "\t"
collection items terminated by ",";

--查询
select movie,category_name
from movie_info
LATERAL VIEW EXPLODE(category) tmpTable as category_name;

--结果
movie   category_name
《疑犯追踪》    悬疑
《疑犯追踪》    动作
《疑犯追踪》    科幻
《疑犯追踪》    剧情
《Lie to me》   悬疑
《Lie to me》   警匪
《Lie to me》   动作
《Lie to me》   心理
《Lie to me》   剧情
《战狼2》       战争
《战狼2》       动作
《战狼2》       灾难
```

#### 窗口函数

| 函数        | 功能                                                         |
| ----------- | ------------------------------------------------------------ |
| OVER()      | 指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变化 |
| CURRENT ROW | 当前行                                                       |
| n PRECEDING | 往前n行数据                                                  |
| n FOLLOWING | 往后n行数据                                                  |
| UNBOUNDED   | 起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING表示到后面的终点 |
| LAG(col,n)  | 往前第n行数据                                                |
| LEAD(col,n) | 往后第n行数据                                                |
| NTILE(n)    | 把有序分区中的行分发到指定数据的组中，各个组有编号，编号从1开始：n必须为int类型。 |

```sql
--准备数据
[hadoop@datanode1 datas]$ vim business.txt
jack,2018-01-01,10
tony,2018-01-02,15
jack,2018-02-03,23
tony,2018-01-04,29
jack,2018-01-05,46
jack,2018-04-06,42
tony,2018-01-07,50
jack,2018-01-08,55
mart,2018-04-08,62
mart,2018-04-09,68
neil,2018-05-10,12
mart,2018-04-11,75
neil,2018-06-12,80
mart,2018-04-13,94

--创建表导入数据
create table business(
name string, 
orderdate string,
cost int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

load data local inpath "/opt/module/datas/business.txt" into table business;

--查询在2018年4月份购买过的顾客及总人数
select name,count(*) over () 
from business 
where substring(orderdate,1,7) = '2018-04' 
group by name;

name    count_window_0
mart    2
jack    2

--查询顾客的购买明细及月购买总额
select name,orderdate,cost,sum(cost) over(partition by month(orderdate)) from business;

--查询顾客的购买明细及月购买总额
select *,sum(cost) over(partition by month(orderdate)) from business;

--查询顾客上次的购买时间
select *,
lag(orderdate,1) over(distribute by name sort by orderdate),
lead(orderdate,1) over(distribute by name sort by orderdate) from business;

select *,
lag(orderdate,1,"2016-12-31") over(distribute by name sort by orderdate)
from business;

--查询前20%时间的订单信息
select *,ntile(5) over(sort by orderdate) gid from business where gid=1;X

select *,ntile(5) over(sort by orderdate) gid from business having gid=1;X

	select *
	from(
	select *,ntile(5) over(sort by orderdate) gid from business
	) t
	where
	gid=1;

select * from (
    select name,orderdate,cost, ntile(5) over(order by orderdate) sorted
    from business
) t
where sorted = 1;
```

#### Rank

| 函数         | 功能                         |
| ------------ | ---------------------------- |
| RANK()       | 排序相同时会重复，总数不会变 |
| DENSE_RANK() | 排序相同时会重复，总数会减少 |
| ROW_NUMBER() | 会根据顺序计算               |



```sql
--数据准备
[hadoop@datanode1 datas]$  vi score.txt
孙悟空,语文,87,
孙悟空,数学,95,
孙悟空,英语,73,
白龙马,语文,94,
白龙马,数学,56,
白龙马,英语,84,
鲁智深,语文,89,
鲁智深,数学,86,
鲁智深,英语,84,
白骨精,语文,79,
白骨精,数学,85,
白骨精,英语,78,

--建表导入数据
create table score(
name string,
subject string, 
score int) 
row format delimited fields terminated by "\t";
load data local inpath '/opt/module/datas/score.txt' into table score;

--结果集
name    subject score   rp      drp     rmp
孙悟空  数学    95      1       1       1
鲁智深  数学    86      2       2       2
白骨精  数学    85      3       3       3
白龙马  数学    56      4       4       4
鲁智深  英语    84      1       1       1
白龙马  英语    84      1       1       2
白骨精  英语    78      3       2       3
孙悟空  英语    73      4       3       4
白龙马  语文    94      1       1       1
鲁智深  语文    89      2       2       2
孙悟空  语文    87      3       3       3
白骨精  语文    79      4       4       4


```

#### 自定义函数

Hive 自带了一些函数，比如：max/min等，但是数量有限，自己可以通过自定义UDF来方便的扩展。

当Hive提供的内置函数无法满足你的业务处理需要时，此时就可以考虑使用用户自定义函数（UDF：user-defined function）。

根据用户自定义函数类别分为以下三种：

（1）UDF（User-Defined-Function） 一进一出

（2）UDAF（User-Defined Aggregation Function） 聚集函数，多进一出  类似于：count/max/min

（3）UDTF（User-Defined Table-Generating Functions）   一进多出  如lateral view explore()

[官方文档地址](https://cwiki.apache.org/confluence/display/Hive/HivePlugins)

编程步骤:

（1）继承org.apache.hadoop.hive.ql.UDF

（2）需要实现evaluate函数；evaluate函数支持重载；

（3）在hive的命令行窗口创建函数

```shell
## 添加jar
add jar jar_path

## 创建function
create [temporary] function [dbname.]function_name AS class_name;

## 在hive的命令行窗口删除函数
Drop [temporary] function [if exists] [dbname.]function_name;
```

​    自定义UDF函数

```xml
<dependencies>
		<!-- https://mvnrepository.com/artifact/org.apache.hive/hive-exec -->
		<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-exec</artifactId>
			<version>1.2.1</version>
		</dependency>
</dependencies>
```

java类
```
import org.apache.hadoop.hive.ql.exec.UDF;

public class Lower extends UDF {

	public String evaluate (final String s) {
		
		if (s == null) {
			return null;
		}
		
		return s.toLowerCase();
	}
}
```

上传上服务器

![kpopw9.png](https://s2.ax1x.com/2019/01/17/kpopw9.png)

添加jar

```sql
hive> add jar /home/hadoop/udf.jar;
Added [/home/hadoop/udf.jar] to class path
Added resources: [/home/hadoop/udf.jar]
--添加关联
hive> add jar /home/hadoop/udf.jar;
Added [/home/hadoop/udf.jar] to class path
Added resources: [/home/hadoop/udf.jar]
hive>  create temporary function mylower as "com.hph.Lower";
OK
Time taken: 0.033 seconds

--调用
hive>  select ename, mylower(ename) lowername from emp limit 2;
OK
ename   lowername
SMITH   smith
ALLEN   allen
```














