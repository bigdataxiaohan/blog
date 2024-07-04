---
title: oozie
date: 2019-01-10 09:39:09
tags: oozie
categories: 大数据
---

## 简介

Oozie英文翻译为：驯象人。一个基于工作流引擎的开源框架，由Cloudera公司贡献给Apache，提供对Hadoop
Mapreduce、Pig Jobs的任务调度与协调。Oozie需要部署到Java Servlet容器中运行。主要用于定时调度任务，多任务可以按照执行的逻辑顺序调度。

## 功能

Oozie是一个管理Hdoop作业（job）的工作流程调度管理系统
Oozie的工作流是一系列动作的直接周期图（DAG）
Oozie协调作业就是通过时间（频率）和有效数据触发当前的Oozie工作流程
Oozie是Yahoo针对Apache Hadoop开发的一个开源工作流引擎。用于管理和协调运行在Hadoop平台上（包括：HDFS、Pig和MapReduce）的Jobs。Oozie是专为雅虎的全球大规模复杂工作流程和数据管道而设计
Oozie围绕两个核心：工作流和协调器，前者定义任务的拓扑和执行逻辑，后者负责工作流的依赖和触发

## 模块

1. Workflow：顺序执行流程节点，支持fork（分支多个节点），join（合并多个节点为一个）

2. Coordinator：定时触发workflow

3. Bundle Job：绑定多个Coordinator

### 常用节点

1. 控制流节点（Control Flow Nodes）：控制流节点一般都是定义在工作流开始或者结束的位置，比如start,end,kill等。以及提供工作流的执行路径机制，如decision，fork，join等。

2. 动作节点（Action  Nodes）：负责执行具体动作的节点，比如：拷贝文件，执行某个Shell脚本等等。

## 部署

所需软件链接  链接：链接：https://pan.baidu.com/s/18_iOFGL06g7_Ye-mZZRwag   提取码：qlbu 


###  部署 Hadoop

这里不详细介绍，请查阅Hadoop安装，这里用的是Clouder公司的CDH版本的Hadop。

### 修改配置

#### core-site.xml

```xml
[hadoop@datanode1 hadoop]$ vim core-site.xml
<configuration>
        <!-- 指定HDFS中NameNode的地址 -->
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://datanode1:9000</value>
        </property>
        <!-- 指定hadoop运行时产生文件的存储目录 -->
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/opt/module/cdh/hadoop-2.5.0-cdh5.3.6/data</value>
        </property>
         <property>
                <name>fs.trash.interval </name>
                <value>60</value>
        </property>
        <!-- Oozie Server的Hostname -->
        <property>
                <name>hadoop.proxyuser.hadoop.hosts</name>
                <value>*</value>
        </property>

        <!-- 允许被Oozie代理的用户组 -->
        <property>
                <name>hadoop.proxyuser.hadoop.groups</name>
                <value>*</value>
        </property>
</configuration>
```

hadoop.proxyuser.admin.hosts类似属性中的hadoop用户替换成你的hadoop用户。因为我的用户名就是hadoop

####  yarn-site.xml

```xml
[hadoop@datanode1 hadoop]$ vim yarn-site.xml
<configuration>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>

        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>datanode2</value>
        </property>

        <property>
                <name>yarn.log-aggregation-enable</name>
                <value>true</value>
        </property>

        <property>
                <name>yarn.log-aggregation.retain-seconds</name>
                <value>86400</value>
        </property>

        <!-- 任务历史服务 -->
        <property>
                <name>yarn.log.server.url</name>
                <value>http://datanode1:19888/jobhistory/logs/</value>
        </property>
</configuration>

```

#### mapred-site.xml

```xml
<configuration>
        <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <!-- 配置 MapReduce JobHistory Server 地址 ，默认端口10020 -->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>datanode1:10020</value>
    </property>
    <!-- 配置 MapReduce JobHistory Server web ui 地址， 默认端口19888 -->
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>datanode1:19888</value>
    </property>
</configuration>
```

不要忘记同步到其他集群 然后namenode -for mate 执行初始化

### 部署 Oozie

#### oozie根目录下解压hadooplibs

```shell
 tar -zxf oozie-hadooplibs-4.0.0-cdh5.3.6.tar.gz -C ../
```

#### 在Oozie根目录下创建libext目录

```shell
mkdir libext/
```

#### 拷贝依赖Jar包

```shell
cp -ra hadooplibs/hadooplib-2.5.0-cdh5.3.6.oozie-4.0.0-cdh5.3.6/* libext/
```

####  上传Mysql驱动包到libext目录下

#### 上传ext-2.2.zip拷贝到libext目录下

#### 修改oozie-site.xml

```
属性：oozie.service.JPAService.jdbc.driver
属性值：com.mysql.jdbc.Driver
解释：JDBC的驱动

属性：oozie.service.JPAService.jdbc.url
属性值：jdbc:mysql://datanode1:3306/oozie
解释：oozie所需的数据库地址

属性：oozie.service.JPAService.jdbc.username
属性值：root
解释：数据库用户名

属性：oozie.service.JPAService.jdbc.password
属性值：123456
解释：数据库密码

属性：oozie.service.HadoopAccessorService.hadoop.configurations
属性值：*=/opt/module/cdh/hadoop-2.5.0-cdh5.3.6/etc/hadoop
解释：让Oozie引用Hadoop的配置文件
```

#### 在Mysql中创建Oozie的数据库

``` shell
mysql -uroot -p123456
mysql> create database oozie;
```

#### 初始化Oozie

```shell
 bin/oozie-setup.sh sharelib create -fs hdfs://datanode1:9000 -locallib oozie-sharelib-4.0.0-cdh5.3.6-yarn.tar.gz
```

###### 创建oozie.sql文件

```
bin/oozie-setup.sh db create -run -sqlfile oozie.sql
```

###### 打包项目，生成war包

```
bin/oozie-setup.sh prepare-war
```

需要zip命令 最小化安装可能需要

#### Oozie服务

```shell
 bin/oozied.sh start
//如需正常关闭Oozie服务，请使用：
 bin/oozied.sh stop
```

##  Web页面

![FOQD2T.png](https://s2.ax1x.com/2019/01/10/FOQD2T.png)

## Oozie任务

### 调度shell

1.解压官方模板

```shell
tar -zxf oozie-examples.tar.gz
```

2.创建工作目录

```shell
mkdir oozie-apps/
```

3.拷贝任务模板

```shell
cp -r examples/apps/shell/ oozie-apps/
```

4.shell脚本

```shell
#!/bin/bash
i=1
mkdir /home/hadoop/oozie-test1
cd /home/hadoop/oozie-test1
for(( i=1;i<=100;i++ ))
do
 d=$( date +%Y-%m-%d\ %H\:%M\:%S )
 echo "data:$d $i">>/home/hadoop/oozie-test1/logs.log
done
```

5.job.properties

```
nameNode=hdfs://datanode1:9000
jobTracker=datanode2:8032
queueName=shell
examplesRoot=oozie-apps

oozie.wf.application.path=${nameNode}/user/${user.name}/${examplesRoot}/shell
EXEC=p1.sh

```

6.workflow.xml

```xml
<workflow-app xmlns="uri:oozie:workflow:0.4" name="shell-wf">
    <start to="shell-node"/>
    <action name="shell-node">
        <shell xmlns="uri:oozie:shell-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <exec>${EXEC}</exec>
            <file>/user/hadoop/oozie-apps/shell/${EXEC}#${EXEC}</file>
            <capture-output/>
        </shell>
        <ok to="end"/>
        <error to="fail"/>
    </action>
    <decision name="check-output">
        <switch>
            <case to="end">
                ${wf:actionData('shell-node')['my_output'] eq 'Hello Oozie'}
            </case>
            <default to="fail-output"/>
        </switch>
    </decision>
    <kill name="fail">
        <message>Shell action failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <kill name="fail-output">
        <message>Incorrect output, expected [Hello Oozie] but was [${wf:actionData('shell-node')['my_output']}]</message>
    </kill>
    <end name="end"/>
</workflow-app>
```

7.上传任务配置

```
/opt/module/cdh/hadoop-2.5.0-cdh5.3.6/bin/hdfs dfs -put  -f  oozie-apps/ /user/hadoop
```

8.执行任务

```shell
 bin/oozie job -oozie http://datanode1:11000/oozie -config oozie-apps/shell/job.properties -run
```

9.杀死任务

```
bin/oozie job -oozie http://datanode1:11000/oozie -kill 0000004-170425105153692-oozie-z-W
```



![FOlpQS.png](https://s2.ax1x.com/2019/01/10/FOlpQS.png)

### 调度逻辑shell

在原有的基础上进行适当修改

1.job.properties

```properties
nameNode=hdfs://datanode1:9000
jobTracker=datanode2:8032
queueName=shell
examplesRoot=oozie-apps

oozie.wf.application.path=${nameNode}/user/${user.name}/${examplesRoot}/shell
EXEC1=p1.sh
EXEC2=p2.sh
```

2.脚本  p1.sh  

```properties
#!/bin/bash               
mkdir /home/hadoop/Oozie2_test_p1                 
cd /home/hadoop/Oozie2_test_p1
i=1
for(( i=1;i<=100;i++ ))
do
 d=$( date +%Y-%m-%d\ %H\:%M\:%S )
 echo "data:$d $i">>/home/hadoop/Oozie2_test_p1/Oozie2_p1.log
done
```

2.脚本  p2.sh

```shell
#!/bin/bash
mkdir /home/hadoop/Oozie2_test_p1
cd /home/hadoop/Oozie2_test_p1
i=1
for(( i=1;i<=100;i++ ))
do
 d=$( date +%Y-%m-%d\ %H\:%M\:%S )
 echo "data:$d $i">>/home/hadoop/Oozie2_test_p1/Oozie2_p1.log
done
```

3.workflow.xml

```xml
<workflow-app xmlns="uri:oozie:workflow:0.4" name="shell-wf">
    <start to="shell-node"/>
    <action name="shell-node">
        <shell xmlns="uri:oozie:shell-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <exec>${EXEC1}</exec>
            <file>/user/hadoop/oozie-apps/shell/${EXEC1}#${EXEC1}</file>
            <capture-output/>
        </shell>
        <ok to="p2-shell-node"/>
        <error to="fail"/>
    </action>

    <action name="p2-shell-node">
        <shell xmlns="uri:oozie:shell-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <exec>${EXEC2}</exec>
            <file>/user/hadoop/oozie-apps/shell/${EXEC2}#${EXEC2}</file>
            <!-- <argument>my_output=Hello Oozie</argument>-->
            <capture-output/>
        </shell>
        <ok to="end"/>
        <error to="fail"/>
    </action>
    
    <decision name="check-output">
        <switch>
            <case to="end">
                ${wf:actionData('shell-node')['my_output'] eq 'Hello Oozie'}
            </case>
            <default to="fail-output"/>
        </switch>
    </decision>
    <kill name="fail">
        <message>Shell action failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <kill name="fail-output">
        <message>Incorrect output, expected [Hello Oozie] but was [${wf:actionData('shell-node')['my_output']}]</message>
    </kill>
    <end name="end"/>
</workflow-app>
```

### 调度MapReduce

前提：确定YARN可用

1.拷贝官方模板到oozie-apps

```
[hadoop@datanode1 lib]$ cp /opt/module/cdh/hadoop-2.5.0-cdh5.3.6/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.0-cdh5.3.6.jar ./
```

2.配置job.properties

```properties
nameNode=hdfs://datanode1:9000
jobTracker=datanode2:8032
queueName=map-reduce
examplesRoot=oozie-apps

oozie.wf.application.path=${nameNode}/user/${user.name}/${examplesRoot}/map-reduce/workflow.xml
outputDir=/output
```

3.workflow.xml

```xml
<workflow-app xmlns="uri:oozie:workflow:0.2" name="map-reduce-wf">
    <start to="mr-node"/>
    <action name="mr-node">
        <map-reduce>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <prepare>
                <delete path="/output"/>
            </prepare>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            <!-- 配置调度MR任务时，使用新的API -->
                <property>
                    <name>mapred.mapper.new-api</name>
                    <value>true</value>
                </property>

                <property>
                    <name>mapred.reducer.new-api</name>
                    <value>true</value>
                </property>
            <!-- 指定Job Key输出类型 -->
                <property>
                    <name>mapreduce.job.output.key.class</name>
                    <value>org.apache.hadoop.io.Text</value>
                </property>
            <!-- 指定Job Value输出类型 -->
                <property>
                    <name>mapreduce.job.output.value.class</name>
                    <value>org.apache.hadoop.io.IntWritable</value>
                </property>
            <!-- 指定Map类 -->
                <property>
                    <name>mapreduce.job.map.class</name>
                    <value>org.apache.hadoop.examples.WordCount$TokenizerMapper</value>
                </property>
             <!-- 指定Reduce类 -->
                <property>
                    <name>mapreduce.job.reduce.class</name>
                    <value>org.apache.hadoop.examples.WordCount$IntSumReducer</value>
                </property>
                <property>
                    <name>mapred.map.tasks</name>
                    <value>1</value>
                </property>
                <property>
                    <name>mapred.input.dir</name>
                    <value>/input</value>
                </property>
                <property>
                    <name>mapred.output.dir</name>
                    <value>/_output</value>
                </property>
            </configuration>
        </map-reduce>
        <ok to="end"/>
        <error to="fail"/>
    </action>
    <kill name="fail">
        <message>Map/Reduce failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name="end"/>
</workflow-app>
```

4.拷贝jar包

```shell
[hadoop@datanode1 lib]$ cp /opt/module/cdh/hadoop-2.5.0-cdh5.3.6/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.0-cdh5.3.6.jar ./
```

5.上传任务配置

```shell
/opt/module/cdh/hadoop-2.5.0-cdh5.3.6/bin/hdfs dfs -put -f oozie-apps /user/hadoop/oozie-apps
```

6.执行任务

```shell
[hadoop@datanode1 oozie-4.0.0-cdh5.3.6]$  bin/oozie job -oozie http://datanode1:11000/oozie -config oozie-apps/map-reduce/job.properties -run
```

7.查看结果

``` 
[hadoop@datanode1 module]$ /opt/module/cdh/hadoop-2.5.0-cdh5.3.6/bin/hdfs dfs -cat /input/*.txt
19/01/10 19:13:37 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
I
Love
Hadoop
and
Sopark
I
Love
BigData
and
AI
[hadoop@datanode1 module]$ /opt/module/cdh/hadoop-2.5.0-cdh5.3.6/bin/hdfs dfs -cat /_output/p*
19/01/10 19:13:08 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
AI      1
BigData 1
Hadoop  1
I       2
Love    2
Sopark  1
and     2
```

### 调度定时任务/循环任务

前提：

```shell
##检查系统当前时区： 
 date -R
##注意这里，如果显示的时区不是+0800，你可以删除localtime文件夹后，再关联一个正确时区的链接过去，命令如下：
 rm -rf /etc/localtime
 ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
```

ntp配置

```
vim /etc/ntp.conf
```

主机配置
[![F0EkqI.png](https://s1.ax1x.com/2018/12/17/F0EkqI.png)](https://s1.ax1x.com/2018/12/17/F0EkqI.png)

从机配置
[![F0mrcD.md.png](https://s1.ax1x.com/2018/12/17/F0mrcD.md.png)](https://s1.ax1x.com/2018/12/17/F0mrcD.md.png)

从节点同步时间

```shell
service ntpd restart
chkconfig ntpd on  # 开机启动
ntpdate -u datanode1
crontab -e
* */1 * * * /usr/sbin/ntpdate datanode1     #每一小时同步一次  注意 要用root创建
```

1.配置oozie-site.xml文件

```
属性：oozie.processing.timezone
属性值：GMT+0800
解释：修改时区为东八区区时
```

2.修改js框架代码

```js
 vi /opt/module/cdh/oozie-4.0.0-cdh5.3.6/oozie-server/webapps/oozie/oozie-console.js
修改如下：
function getTimeZone() {
    Ext.state.Manager.setProvider(new Ext.state.CookieProvider());
    return Ext.state.Manager.get("TimezoneId","GMT+0800");
}
```

3.重启oozie服务，并重启浏览器（一定要注意清除缓存）

```
bin/oozied.sh stop
bin/oozied.sh start
```

4.拷贝官方模板配置定时任务

```
cp -r examples/apps/cron/ oozie-apps/
```

5.修改job.properties

```properties
nameNode=hdfs://datanode1:9000
jobTracker=datanode2:8032
queueName=cronTask
examplesRoot=oozie-apps

oozie.coord.application.path=${nameNode}/user/${user.name}/${examplesRoot}/cron
start=2019-01-10T21:40+0800
end=2019-01-10T22:00+0800
workflowAppUri=${nameNode}/user/${user.name}/${examplesRoot}/cron

EXEC3=p3.sh

```

6.修改coordinator.xml  注意${coord:minutes(5)}的5是最小值不能比5再小了

```xml
<coordinator-app name="cron-coord" frequency="${coord:minutes(5)}" start="${start}" end="${end}" timezone="GMT+0800"
                 xmlns="uri:oozie:coordinator:0.2">
        <action>
        <workflow>
            <app-path>${workflowAppUri}</app-path>
            <configuration>
                <property>
                    <name>jobTracker</name>
                    <value>${jobTracker}</value>
                </property>
                <property>
                    <name>nameNode</name>
                    <value>${nameNode}</value>
                </property>
                <property>
                    <name>queueName</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
        </workflow>
    </action>
</coordinator-app>
```

7.创建脚本

```shell
#!/bin/bash
d=$( date +%Y-%m-%d\ %H\:%M\:%S )
echo "data:$d $i">>/home/hadoop/Oozie3_p3.log
```

8.修改

```xml
<workflow-app xmlns="uri:oozie:workflow:0.5" name="one-op-wf">
<start to="p3-shell-node"/>
  <action name="p3-shell-node">
      <shell xmlns="uri:oozie:shell-action:0.2">
          <job-tracker>${jobTracker}</job-tracker>
          <name-node>${nameNode}</name-node>
          <configuration>
              <property>
                  <name>mapred.job.queue.name</name>
                  <value>${queueName}</value>
              </property>
          </configuration>
          <exec>${EXEC3}</exec>
          <file>/user/hadoop/oozie-apps/cron/${EXEC3}#${EXEC3}</file>
          <!-- <argument>my_output=Hello Oozie</argument>-->
          <capture-output/>
      </shell>
      <ok to="end"/>
      <error to="fail"/>
  </action>
<kill name="fail">
    <message>Shell action failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
</kill>
<kill name="fail-output">
    <message>Incorrect output, expected [Hello Oozie] but was [${wf:actionData('shell-node')['my_output']}]</message>
</kill>
<end name="end"/>
</workflow-app>
```

9.提交配置

```shell
/opt/module/cdh/hadoop-2.5.0-cdh5.3.6/bin/hdfs dfs -put oozie-apps/cron/ /user/hadoop/oozie-apps
```

10.提交任务

```shell
bin/oozie job -oozie http://datanode1:11000/oozie -config oozie-apps/cron/job.properties -run
```

![FOxtXD.png](https://s2.ax1x.com/2019/01/10/FOxtXD.png)





















