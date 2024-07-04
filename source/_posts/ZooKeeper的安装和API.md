---
title: ZooKeeper的安装和API
date: 2019-01-06 13:43:27
tags: Zookeeper
categories: 大数据
---

## 安装教程

在datanode1、datanode2和datanode3三个节点上部署Zookeeper。

### 步骤

1. 解压zookeeper安装包到/opt/module/目录下

```shell
tar -zxvf zookeeper-3.4.10.tar.gz -C /opt/module/
```

2. /opt/module/zookeeper-3.4.10/这个目录下创建zkData

```shell
mkdir -p zkData
```

3. 重命名/opt/module/zookeeper-3.4.10/conf这个目录下的zoo_sample.cfg为zoo.cfg

```shell
mv zoo_sample.cfg zoo.cfg
```

4. 配置zoo.cfg文件

```shell
#######################cluster##########################
server.2=datanode1:2888:3888
server.3=datanode2:2888:3888
server.4=datanode3:2888:3888
```

server.A=B:C:D。

A是一个数字，表示这个是第几号服务器；

B是这个服务器的ip地址或者主机名；

C是这个服务器与集群中的Leader服务器交换信息的端口；

D是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。

5. 在/opt/module/zookeeper-3.4.10/zkData目录下创建一个myid的文件

```shell
touch myid
```

6. 编辑myid文件,各个节点的值根据配置文件zoo.cfg来

7. 拷贝配置好的zookeeper到其他机器上并分别修改myid文件中内容为3、4

```shell
touch myid
```

8. 启动脚本

```shell
#!/bin/sh
echo "starting zookeeper server..."
hosts="datanode1 datanode2 datanode3" 
for host in $hosts
do
  ssh $host  "source /etc/profile; /opt/module/zookeeper-3.4.10/bin/zkServer.sh start"
done
```

停止脚本换成stop即可

9. 查看zhua状态

```shell
zkServer.sh status
```

### 客户端命令行操作

| 命令基本语法      | 功能描述                                               |
| ----------------- | ------------------------------------------------------ |
| help              | 显示所有操作命令                                       |
| ls   path [watch] | 使用 ls 命令来查看当前znode中所包含的内容              |
| ls2 path [watch]  | 查看当前节点数据并能看到更新次数等数据                 |
| create            | 普通创建   -s  含有序列   -e  临时（重启或者超时消失） |
| get path [watch]  | 获得节点的值                                           |
| set               | 设置节点的具体值                                       |
| stat              | 查看节点状态                                           |
| delete            | 删除节点                                               |
| rmr               | 递归删除节点                                           |

## Shell命令

### 启动客户端

```shell
bin/zkCli.sh
```

### 显示所有操作命令

```shell
[zk: localhost:2181(CONNECTED) 0] help
ZooKeeper -server host:port cmd args
        connect host:port
        get path [watch]
        ls path [watch]
        set path data [version]
        rmr path
        delquota [-n|-b] path
        quit
        printwatches on|off
        create [-s] [-e] path data acl
        stat path [watch]
        close
        ls2 path [watch]
        history
        listquota path
        setAcl path acl
        getAcl path
        sync path
        redo cmdno
        addauth scheme auth
        delete path [version]
        setquota -n|-b val path
```

### 查看当前znode中所包含的内容

```shell
[zk: localhost:2181(CONNECTED) 1] ls /
[isr_change_notification, test, hbase, zookeeper, admin, consumers, cluster, config, latest_producer_id_block, kafka-manager, brokers, controller_epoch]
```

### 查看当前节点数据并能看到更新次数等数据

```shell
[zk: localhost:2181(CONNECTED) 2]  ls2 /
[isr_change_notification, test, hbase, zookeeper, admin, consumers, cluster, config, latest_producer_id_block, kafka-manager, brokers, controller_epoch]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x1200000104
cversion = 100
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 12
```

### 创建普通节点

```shell
[zk: localhost:2181(CONNECTED) 3]  create /app1 "hello app1"
Created /app1
[zk: localhost:2181(CONNECTED) 4] create /app1/server101 "192.168.1.101"
Created /app1/server101
```

### 获得节点的值

```shell
[zk: localhost:2181(CONNECTED) 5] get /app1
hello app1
cZxid = 0x120000010a
ctime = Sat Jan 05 13:34:15 CST 2019
mZxid = 0x120000010a
mtime = Sat Jan 05 13:34:15 CST 2019
pZxid = 0x120000010b
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 10
numChildren = 1
[zk: localhost:2181(CONNECTED) 6] get /app1/server101
192.168.1.101
cZxid = 0x120000010b
ctime = Sat Jan 05 13:34:46 CST 2019
mZxid = 0x120000010b
mtime = Sat Jan 05 13:34:46 CST 2019
pZxid = 0x120000010b
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 13
numChildren = 0
```

### 创建短暂节点

```shell
[zk: localhost:2181(CONNECTED) 7] create -e /app-emphemeral 8888
Created /app-emphemeral
## 在当前客户端是能查看到的
[zk: localhost:2181(CONNECTED) 8]  ls /
[app1, isr_change_notification, test, hbase, zookeeper, admin, consumers, cluster, config, latest_producer_id_block, kafka-manager, app-emphemeral, brokers, controller_epoch]
## 退出当前客户端然后再重启客户端
[zk: localhost:2181(CONNECTED) 9] quit
Quitting...
2019-01-06 17:43:54,558 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x1681924b94d0014 closed
2019-01-06 17:43:54,564 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@519] - EventThread shut down for session: 0x1681924b94d0014
## 重启客户端
[hadoop@datanode1 bin]$ ./zkCli.sh
....
2019-01-06 17:46:04,711 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x1681924b94d0015, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
## 再次查看根目录下短暂节点已经删除

[zk: localhost:2181(CONNECTED) 0] ls /
[app1, isr_change_notification, test, hbase, zookeeper, admin, consumers, cluster, config, latest_producer_id_block, kafka-manager, brokers, controller_epoch]
```

### 创建带序号的节点

```shell
## 先创建一个普通的根节点app2
[zk: localhost:2181(CONNECTED) 1] create /app2 "app2"
Created /app2
## 创建带序号的节点
[zk: localhost:2181(CONNECTED) 2] create /app2 "app2"
[zk: localhost:2181(CONNECTED) 3] create -s /app2/aa 888
Created /app2/aa0000000000
[zk: localhost:2181(CONNECTED) 4] create -s /app2/bb 888
Created /app2/bb0000000001
[zk: localhost:2181(CONNECTED) 5] create -s /app2/cc 888
Created /app2/cc0000000002
##如果原节点下有1个节点，则再排序时从1开始，以此类推。
[zk: localhost:2181(CONNECTED) 6] create -s /app1/aa 888
Created /app1/aa0000000001
```

### 修改节点数据值

```shell
[zk: localhost:2181(CONNECTED) 8] set /app1 999
cZxid = 0x120000010a
ctime = Sat Jan 05 13:34:15 CST 2019
mZxid = 0x1200000116
mtime = Sat Jan 05 14:12:56 CST 2019
pZxid = 0x1200000114
cversion = 3
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 3
```

### 节点的值变化监听

在datanode1主机上注册监听/app1节点数据变化

```shell
[zk: localhost:2181(CONNECTED) 9] get /app1 watch
999
cZxid = 0x120000010a
ctime = Sat Jan 05 13:34:15 CST 2019
mZxid = 0x1200000116
mtime = Sat Jan 05 14:12:56 CST 2019
pZxid = 0x1200000114
cversion = 3
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 3
```

在datanode2主机上修改/app1节点的数据

```shell
[zk: localhost:2181(CONNECTED) 0] set /app1  777
cZxid = 0x120000010a
ctime = Sat Jan 05 13:34:15 CST 2019
mZxid = 0x1200000118
mtime = Sat Jan 05 14:14:43 CST 2019
pZxid = 0x1200000114
cversion = 3
dataVersion = 2
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 3
```

datanode1主机上的变化

```shell
[zk: localhost:2181(CONNECTED) 10]
WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/app1
```

### 节点的子节点变化监听（路径变化）

在datanode1主机上注册监听/app1节点的子节点变化

```shell
[zk: localhost:2181(CONNECTED) 0] ls /app1 watch
[aa0000000001, server101, cc0000000002]
```

在datanode2主机/app1节点上创建子节点

```shell
[zk: localhost:2181(CONNECTED) 1] create /app1/bb 666
Created /app1/bb
```

观察datanode1主机收到子节点变化的监听

```shell
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/app1
```

### 删除节点

```shell
[zk: localhost:2181(CONNECTED) 2] delete /app1/bb
```

### 递归删除节点

```
[zk: localhost:2181(CONNECTED) 3] rmr /app2
```

### 查看节点状态

```
[zk: localhost:2181(CONNECTED) 1]  stat /app1
cZxid = 0x120000010a
ctime = Sat Jan 05 13:34:15 CST 2019
mZxid = 0x1200000118
mtime = Sat Jan 05 14:14:43 CST 2019
pZxid = 0x120000011d
cversion = 5
dataVersion = 2
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 3
```

## API应用

### IDEA环境搭建

1. 创建一个Maven工程

2. 添加pom文件

    ```xml
    	<dependencies>
    		<dependency>
    			<groupId>junit</groupId>
    			<artifactId>junit</artifactId>
    			<version>RELEASE</version>
    		</dependency>
    		<dependency>
    			<groupId>org.apache.logging.log4j</groupId>
    			<artifactId>log4j-core</artifactId>
    			<version>2.8.2</version>
    		</dependency>
    		<!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
    		<dependency>
    			<groupId>org.apache.zookeeper</groupId>
    			<artifactId>zookeeper</artifactId>
    			<version>3.4.10</version>
    		</dependency>
    	</dependencies>
    ```

### log4j.propertie

```properties
log4j.rootLogger=INFO, stdout  
log4j.appender.stdout=org.apache.log4j.ConsoleAppender  
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout  
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n  
log4j.appender.logfile=org.apache.log4j.FileAppender  
log4j.appender.logfile.File=target/spring.log  
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout  
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n  
```

### 创建ZooKeeper客户端

```java
public class ZKDemo {
    private String connect = "datanode1:2181,datanode2:2181,datanode3:2181";
    private int timeout = 2000;
    private ZooKeeper zooKeeper = null;

    //获取Zookeeper的客户端
    @Before
    public void getClient() throws Exception {
        zooKeeper = new ZooKeeper(connect, timeout, new Watcher() {
            //接收到Zookeeper发来的通知以后做出的处理措施(自己处理的业务逻辑)
            @Override
            public void process(WatchedEvent watchedEvent) {
                System.out.println(watchedEvent.getType() + "-----" + watchedEvent.getPath());
            }
        });
    }
```

### 创建子节点

```java
    //创建节点
    @Test
    public void testCreate() throws KeeperException, InterruptedException {
     // 参数1：要创建的节点的路径； 参数2：节点数据 ； 参数3：节点权限 ；参数4：节点的类型
        String path = zooKeeper.create(
                "/cainiaoqingfeng",
                "bigtadaLearing".getBytes(),
                ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.PERSISTENT);
        System.out.println(path);
    }
```

```shell
[zk: localhost:2181(CONNECTED) 2] ls /
[test, cainiaoqingfeng, consumers, latest_producer_id_block, controller_epoch, app2, app1, isr_change_notification, hbase, admin, zookeeper, config, cluster, kafka-manager, brokers]
```

### 判断节点是否存在

```java
   public void testExist() throws KeeperException, InterruptedException {
        Stat stat = zooKeeper.exists("/cainiaoqingfeng", false);
        System.out.println(stat==null?"not exist":"exist");
    }
```

### 循环监听

```java
      try {
                    List<String> children = zooKeeper.getChildren("/cainiaoqingfeng", true);
                    for (String child : children) {
                        System.out.println(child);
                    }

                } catch (Exception e) {
                    e.printStackTrace();
                }
//在创建Zookeeper客户端的代码中加此以上代码
```

### 改变节点的内容

```java
    public void testSet() throws KeeperException, InterruptedException {
        Stat stat = zooKeeper.setData("/cainiaoqingfeng/bigdata", "I love bigdata".getBytes(), -1);

    }
```

```shell
[zk: localhost:2181(CONNECTED) 20] get /cainiaoqingfeng/bigdata
I love bigdata
cZxid = 0x1200000126
ctime = Sat Jan 05 16:11:19 CST 2019
mZxid = 0x1200000136
mtime = Sat Jan 05 16:27:51 CST 2019
pZxid = 0x1200000126
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 14
```



## 监听器原理

![FHz0gg.png](https://s2.ax1x.com/2019/01/06/FHz0gg.png)

### 过程

1. 先要有一个main()线程
2. 在main线程中创建Zookeeper客户端，这时就会创建两个线程，一个负责网络连接通信（connet），一个负责监听（listener）。
3. 通过connect线程将注册的监听事件发送给Zookeeper。
4. 在Zookeeper的注册监听器列表中将注册的监听事件添加到列表中。
5. Zookeeper监听到有数据或路径变化，就会将这个消息发送给listener线程。
6. listener线程内部调用了process（）方法。

### 常见监听

2．常见的监听

（1）监听节点数据的变化：

```shell
get path [watch]
```



（2）监听子节点增减的变化

```shell
ls path [watch]
```

### 写数据

![FHzor9.png](https://s2.ax1x.com/2019/01/06/FHzor9.png)

ZooKeeper 的写数据流程主要分为以下几步，如图所示：

#### 流程

1. 比如 Client 向 ZooKeeper 的 Server1 上写数据，发送一个写请求。
2. 如果Server1不是Leader，那么Server1 会把接受到的请求进一步转发给Leader，因为每个ZooKeeper的Server里面有一个是Leader。这个Leader 会将写请求广播给各个Server，比如Server1和Server2， 各个Server写成功后就会通知Leader。
3. 当Leader收到大多数 Server 数据写成功了，那么就说明数据写成功了。如果这里三个节点的话，只要有两个节点数据写成功了，那么就认为数据写成功了。写成功之后，Leader会告诉Server1数据写成功了。
4. Server1会进一步通知 Client 数据写成功了，这时就认为整个写操作成功。ZooKeeper 整个写数据流程就是这样的。

## 服务器节点动态上下线

![FbSLes.png](https://s2.ax1x.com/2019/01/06/FbSLes.png)

### ZkServer

```java
import org.apache.zookeeper.*;

import java.io.IOException;

public class zkServer {
    private static String connect = "datanode1:2181,datanode2:2181,datanode3:2181";  //写自己的集群的ip和端口
    private static int timeout = 2000;
    private static ZooKeeper zooKeeper = null;
    private static String parentPahth = "/servers/";

    public static void main(String[] args) throws IOException, KeeperException, InterruptedException {
        //获取Zookeeper的客户端
        getClient();
        //启动注册
        registsServer(args[0]);
        //业务逻辑
        business(args[0]);
    }

    private static void business(String hostname) throws InterruptedException {
        System.out.println(hostname+"  is working...");
        Thread.sleep(Long.MAX_VALUE);


    }

    private static void registsServer(String hostname) throws KeeperException, InterruptedException {
        //创建临时节点
        String path = zooKeeper.create(parentPahth + "server",
                hostname.getBytes(),
                ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.EPHEMERAL_SEQUENTIAL);
        System.out.println(hostname+"  is online  "+path);
        Thread.sleep(Long.MAX_VALUE);
    }


    private static void getClient() throws IOException {
        zooKeeper = new ZooKeeper(connect, timeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                System.out.println(watchedEvent.getType() + "**********" + watchedEvent.getPath());
            }
        });
    }
}

```

### ZkClient

```java
import org.apache.zookeeper.*;

import java.io.IOException;

public class zkServer {
    private static String connect = "datanode1:2181,datanode2:2181,datanode3:2181";  //写自己的集群的ip和端口
    private static int timeout = 2000;
    private static ZooKeeper zooKeeper = null;
    private static String parentPahth = "/servers/";

    public static void main(String[] args) throws IOException, KeeperException, InterruptedException {
        //获取Zookeeper的客户端
        getClient();
        //启动注册
        registsServer(args[0]);
        //业务逻辑
        business(args[0]);
    }

    private static void business(String hostname) throws InterruptedException {
        System.out.println(hostname+"  is working...");
        Thread.sleep(Long.MAX_VALUE);


    }

    private static void registsServer(String hostname) throws KeeperException, InterruptedException {
        //创建临时节点
        String path = zooKeeper.create(parentPahth + "server",
                hostname.getBytes(),
                ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.EPHEMERAL_SEQUENTIAL);
        System.out.println(hostname+"  is online  "+path);
        Thread.sleep(Long.MAX_VALUE);
    }


    private static void getClient() throws IOException {
        zooKeeper = new ZooKeeper(connect, timeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                System.out.println(watchedEvent.getType() + "**********" + watchedEvent.getPath());
            }
        });
    }
}

```

![FbP57j.png](https://s2.ax1x.com/2019/01/06/FbP57j.png)



![FbPTNn.png](https://s2.ax1x.com/2019/01/06/FbPTNn.png)

​															参考资料:尚硅谷Zookeeper











