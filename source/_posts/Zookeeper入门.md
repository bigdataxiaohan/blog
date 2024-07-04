---
title: Zookeeper入门
date: 2019-01-05 13:24:47
tags: Zookeeper
categories: 大数据
---

## Zookeeper 的简介

Zookeeper是一个开源的分布式的，一个针对大型分布式系统的可靠协调系统的Apache项目。

目标是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户；

ZooKeeper已经成为Hadoop生态系统的基础组件;功能包括：配置服务、名字服务、分布式同步、组服务等

从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应，从而实现集群中类似Master/Slave管理模式

Zookeeper=文件系统+通知机制

## 特点

1. Zookeeper：一个领导者（leader），多个跟随者（follower）组成的集群。
2. Leader负责进行投票的发起和决议，更新系统状态
3. Follower用于接收客户请求并向客户端返回结果，在选举Leader过程中参与投票
4. 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。
5. 全局数据一致：每个server保存一份相同的数据副本，client无论连接到哪个server，数据都是一致的。
6. 更新请求顺序进行，来自同一个client的更新请求按其发送顺序依次执行。
7. 数据更新原子性，一次数据更新要么成功，要么失败。
8. 实时性，在一定时间范围内，client能读到最新数据。

## 特性

1. 简单:Zookeeper的核心是一个简单的文件系统,文件系统提供一些简单的操作和抽象操作比如排序和通知.
2. 富有表现力的:Zookeeper的基本操作是一组丰富的构建(building block),可以用于实现多种协调数据结构和协议如：分布式队列、分布式锁、领导选举(leader election)。
3. 高可用性：Zookeeper是运行在一组机器上的，负责机器资源和进程管理。设计上有高可用性，因此应用程序完全可以依赖它，Zookeeper可以解决系统中出现单点故障的情况，实现系统的高可靠性，Zookeeper可以构建可靠的应用程序。比如Hadoop的HA。
4. 松耦合交互方式：在Zookeeper支持的交互过程中,参与者不需要彼此了解.如Zookeeper可以被用于实现“数据汇聚(rendezvous)”机制，让进程在不了解其他进程(或者网络状况)的情况下能够彼此发现并进行信息交互，参与的各方甚至可以不必要同时存在，因为一个进程可以在Zookeeper中留下一条记录i信息，在该进程结束后，另一个进程可以在Zookeeper中读取到这条记录信息。
5. 资源库：Zookeeper提供了一个通用协调模式实现方法的开源啊共享库，使得程序员免于编写协调协议程序(这类程序很难编写)，所有人都能够对这个资源库进行修改和改进，这样每个人都能够从中受益。
6. 高性能：对于以写操作为主的工作负载来说：Zookeeper的基准吞吐量已经超过每秒10000个操作；对于常规的以读操作为主的工作负载来说，吞吐量要高出好几倍。

## 数据模型

ZooKeeper数据模型的结构与Unix文件系统很类似，整体上可以看作是一棵树，每个节点称做一个ZNode。Zookeeper拥有一个层次的命名空间，这和分布式的文件系统十分相似。唯一不同的地方是命名空间中的每个节点可以有和他自身或它的子节点相关联的数据。这就好像是一个文件系统只不过文件系统中的文件和还具有目录的功能。另外，指向节点的路径必须使用规范的绝对路径来标识，并且以斜线“/”来分隔。ZooKeeper中不允许使用相对路径

很显然zookeeper集群自身维护了一套数据结构。这个存储结构是一个树形结构，其上的每一个节点，我们称之为”znode”，每一个znode默认能够存储1MB的数据，每个ZNode都可以通过其路径唯一标识。

[![F7iYDJ.png](https://s2.ax1x.com/2019/01/05/F7iYDJ.png)](https://s2.ax1x.com/2019/01/05/F7iYDJ.png)

### Znode

Zookeeper目录树中的每一个节点对应着每一个Znode。每个Znode维护者一个属性结构，它包括数据的版本号(dataVersion)、时间戳(ctime、mtime)的状态信息。ZooKeeper使用节点的这些特性来实现它的特定功能。当Znode数据发生改变时，它的版本号将会增加，当客户端检索数据时，会同时检索数据的版本号，如果一个客户端执行了某个节点的更新或者删除操作，它必须提供要被操作的数据的版本号，如果所提供的版本号与实际不匹配，那么这个操作将会失败。

#### Watches

客户端可以在节点上设置watches(监视器)。当节点的状态发生改变时(数据的增、删、改)将会触发watch对应的操作。当watch被触发时,Zookeeper将会向客户端发送且发送一个通知，因为watch只能触发一次。

#### 数据访问

Zookeeper中的每一个节点上存储的数据需要被原子性的操作：读操作将获取与节点相关的数据，写操作也将替换掉节点相关的所有数据。另外，每一个节点都拥有自己的ACL（访问控制列表），这个列表规定了用户的权限，即限定了用户对目标节点可以执行的操作。

#### 临时节点

Zookeeper的节点分为两种：临时节点和永久节点。节点的类型在创建时即被确定，并且不能该改变。Zookeeper临时节点的生命周期依赖于他们的会话，一旦会话结束，临时节点会被自动删除，当然也可以手动删除。Zookeeper的临时节点不允许拥有子节点，相反，永久节点的生命周期不依赖于会话，并且只有客户端显示执行删除操作的时候，他们才被删除。

#### 顺序节点

唯一性保证：当创建Znode的时候，用户可以请求在Zookeeper的路径末尾添加一个递增的计数。这个计数对于此节点的父节点时唯一的格式为“%010d”(10位数组，没有数值的数据位用0填充，例如0000000001)。当计数值大于2147483647，计数器会溢出。

### 时间

Zookeeper中有多重记录时间的形式，主要包括以下几个属性：

#### Zxid

导致Zookeeper节点状态改变的每一个操作都将使节点接收到一个zxid格式的时间戳，并且这个时间戳使全局有序的，也就是说，每一个对节点的改变都将产生一个唯一的zxid。如果zxid1的值小于zxid2的值，那么zxid1所对应的时间发生在zxid2所对应的事件之前，实际中Zookeeper的每个节点都维护着三个zxid值，分别为：cZxid、mZxid和pZid。cZxid使节点的创建时间所对应的Zxid时间戳，mZxid是节点的修改时间对应的Zxid格式时间戳。

#### 版本号

对节点的每一个操作都将导致这个节点的版本号增加。每个节点维护者三个版本号，他们维护者三个版本号，他们分别是：version(节点数据版本号)、cversion(子节点版本号)、aversion(节点所拥有的ACL版本号)。

### 节点属性结构

| 属性        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| cZxid       | 创建节点时的事务ID                                           |
| ctime       | 创建节点的时间                                               |
| mZxid       | 最后修改节点时的事务ID                                       |
| mtime       | 最后修改节点时的时间                                         |
| pZxid       | 表示该子节点列表最后一次修改的事务ID，添加子节点或删除子节点就会影响子节点列表，但是修改子节点的数据内容则不影响该ID |
| cversion    | 子节点版本号，子节点每次修改版本号加1                        |
| dataversion | 数据版本号，数据每次修改该版本号加1                          |
| aclversion  | 权限版本号，权限每次修改该版本号加1                          |
| dataLength  | 该节点的数据长度                                             |
| numChildren | 该节点拥有子节点的数量                                       |

## 观察触发器

读操作exists、getChildren和getData都被设置了watch，并且这些watch都由写操作来触发：create、delete和setData。ACL操作并不参与到watch中。当watch被触发时，watch事件被生成，它的类型由watch和触发它的操作共同决定，Zookeeper所管理的watch可以分为两类：

1. data watchs：getData和exists负责设置data watch；
2. child watchs：getChildren负责设置child watch

我们可以通过操作返回的数据来设置不同的watch：

1. getData和exists：返回关于节点的数据信息
2. getChildren：返回孩子列表

因此，一个成功的setData操作将触发Znode的数据watch，一个成功的create操作将触发Znode的数据watch，一个成功的delete操作将触发Znode的data wachts以及child watches。

watch由客户端所连接的Zookeeper服务器在本地维护，因此可以非常容易地设置，管理和分派，当客户端连接到另一个新的服务器上时，任何的会话事件都可能触发watch。另外当从服务器断开连接的时候，watch将不会被接收。但是，当一个客户端重新建立连接的时候，任何先前注册过的watch都会被重新注册。

exists操作上的watch，在被监视的Znode创建、删除或数据更新时被触发。

getData操作上的watch，在被监视的Znode删除或数据更新时被触发，在创建时不能被触发，因为只有Znode存在，getData操作才会成功。

getChildren操作上的watch，在被监视的Znode的子节点创建或删除，或是这个Znode自身被删除时被触发。可以通过查看watch事件类型来区分是Znode还是其他的子节点被删除；NodeDelete表示Znode被删除，NodeDeletedChanged表示子节点被删除。

[![F7QBTK.png](https://s2.ax1x.com/2019/01/05/F7QBTK.png)](https://s2.ax1x.com/2019/01/05/F7QBTK.png)

watch事件包括了设计的Znode的路径，因此对于NodeCreated和NodeDeleted事件来说，根据路径就可以简单区分出那个Znode被创建或者时被删除了。为了查询在NodeChildrenChanged事件后产生的新数据，需要调用getData。在两种情况下，Znode可能在获取watch事件或执行读写操作这两种状态下切换，在写应用程序时，必须记住这一点。

Zookeeper的watch实际上需要处理两类事件：

- 连接状态事件(type=None,path=null)

这类事件不需要注册，也不需要我们连续触发，我们只需要处理就行了

- 节点事件

节点的创建，删除数据的修改。他是 one time trigger,我们需要不停的注册触触发，还可能发生事件丢失的情况。

上面2类事件都在Watch中处理，也就是重载的**process(Event event)**

**节点事件的触发，通过函数exists，getData或getChildren来处理这类函数，有双重作用：**

**① 注册触发事件**

**② 函数本身的功能**

函数的本身的功能又可以用异步的回调函数来实现，重载processResult)()过程中处理函数本身的功能。函数还可以指定自己的watch所以每个函数有4个版本。根据自己的需求来选择不同的函数，不同的版本。

## ACL列表

ZooKeeper使用ACL来对Znode进行访问控制。ACL的实现和Unix文件访问许可非常相似：它使用许可位来对一个节点的不同操作进行允许或禁止的权限控制。但是，和标准的Unix许可不同的是，Zookeeper对于用户类别的区分，不知局限于所有者(owner)、组(group)、所有人(world)、三个级别，Zookeeper中，数据节点没有“所有者”的概念。访问者利用id表示自己的身份，并且获得与之相应的不同的访问权限。

注意：

传统的文件系统中，ACL分为两个维度，一个是属组，一个是权限，子目录/文件默认继承父目录的ACL。而在Zookeeper中一个ACL和一个ZooKeeper节点相对应。并且，父节点的ACL与子节点的ACL是相互独立的。也就是说，ACL不能被子节点所继承，父节点所拥有的权限与子节点所用的权限都没有任何关系。

Zookeeper支持可配置的认证机制。它利用一个三元组来定义客户端的访问权限：(scheme:expression, perms) 。其中：

1.scheme：定义了expression的含义。

如：host:host1.corp.com,READ）,标识了一个名为host1.corp.com的主机,有该数据节点的读权限。

2.Perms：标识了操作权限。

如：（ip:19.22.0.0/16, READ）,表示IP地址以19.22开头的主机,有该数据节点的读权限。

Zookeeper的ACL也可以从三个维度来理解：一是，scheme; 二是，user; 三是，permission，通常表示为scheme:id：permissions，如下图所示。

[![F71cqI.png](https://s2.ax1x.com/2019/01/05/F71cqI.png)](https://s2.ax1x.com/2019/01/05/F71cqI.png)

1.world : id格式：anyone。

如：world:anyone代表任何人，zookeeper中对所有人有权限的结点就是属于world:anyone的。

2.auth : 它不需要id。

注：只要是通过authentication的user都有权限，zookeeper支持通过kerberos来进行认证, 也支持username/password形式的认证。

3.digest: id格式：username:BASE64(SHA1(password))。

它需要先通过username:password形式的authentication。

4.ip: id格式：客户机的IP地址。

设置的时候可以设置一个ip段。如：ip:192.168.1.0/16, 表示匹配前16个bit的IP段

5.super: 超级用户模式。

在这种scheme情况下，对应的id拥有超级权限，可以做任何事情

ZooKeeper权限定义如下图所示：

[![F717ss.png](https://s2.ax1x.com/2019/01/05/F717ss.png)](https://s2.ax1x.com/2019/01/05/F717ss.png)

ZooKeeper内置的ACL模式如下图所示：

[![F71OoV.png](https://s2.ax1x.com/2019/01/05/F71OoV.png)](https://s2.ax1x.com/2019/01/05/F71OoV.png)

当会话建立的时候，客户端将会进行自我验证。另外，ZooKeeper Java API支持三种标准的用户权限，它们分别为：

1.ZOO_PEN_ACL_UNSAFE：对于所有的ACL来说都是完全开放的，任何应用程序可以在节点上执行任何操作，比如创建、列出并删除子节点。

2.ZOO_READ_ACL_UNSAFE：对于任意的应用程序来说，仅仅具有读权限。

3.ZOO_CREATOR_ALL_ACL：授予节点创建者所有权限。需要注意的是，设置此权限之前，创建者必须已经通了服务器的认证。

代码实现：

```java
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.ACL;
import org.apache.zookeeper.data.Id;
import org.apache.zookeeper.server.auth.DigestAuthenticationProvider;

import java.io.IOException;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.List;


public class NewDigest {
    public static void main(String[] args) throws NoSuchAlgorithmException, KeeperException, InterruptedException, IOException {

        List acls = new ArrayList<>();
        //添加第一个id,采用用户名密码的形式
        Id id1 = new Id("digest", DigestAuthenticationProvider.generateDigest("admin:admin"));
        ACL acl1 = new ACL(ZooDefs.Perms.ALL, id1);
        acls.add(acl1);

        //添加第二个id,所有用户可获得去权限
        Id id2 = new Id("world", "anyone");
        ACL acl2 = new ACL(ZooDefs.Perms.READ, id2);
        ZooKeeper zk = new ZooKeeper("datanode1:2181,datanode2:2181,datanode3:2181", 200, null);

        zk.addAuthInfo("digest","admin:admin".getBytes());
        zk.create("/test","data".getBytes(),acls,CreateMode.PERSISTENT);
        System.out.println("创建成功");

    }
}
```

## 执行过程

Zookeeper服务可以以以两种模式运行，在单机模式下，只有一个Zookeeper服务器，便于用来测试，但是它没有高可用和性能恢复的保障。在工业界，Zookeeper以复合模式运行在一组键ensemble的集群上。Zookeeper通过复制来获得高可用，同时，只要ensemble中大部分机器运作，就可以提供服务，在2n+1个节点的ensemble中，可以承受n台机器出现故障。

ZooKeeper的思想非常简单：它所需要做的就是保证对Znode树的每一次修改都复制到ensemble中的大部分机器上去。如果机器中的小部分出故障了，那么至少有一台机器将会恢复到最新状态，其他的则保存这副本，直到最终达到最新状态。Zookeeper采用Zab协议，它分为两个阶段，并且可能被无限的重复。

### 领导者选举

在ensemble中的机器要参与一个选择特殊成员的进程，这个成员叫领导者，其他机器脚跟随者。在大部分的跟随者与他们的领导者同步了状态以后，这个阶段才算完成。

当leader崩溃或者leader失去大多数的follower，这时候zk进入**恢复模式**，恢复模式需要重新选举出一个新的leader，让所有的Server都恢复到一个正确的状态。Zk的选举算法有两种：一种是基于**basic paxos**实现的，另外一种是基于**fast paxos**算法实现的。系统默认的选举算法为fast paxos。先介绍basic paxos流程：

1.选举线程由**当前Server**发起选举的线程担任，其主要功能是对投票结果进行统计，并选出推荐的Server；

2.选举线程首先向所有Server发起一次**询问**(包括自己)；

3 .选举线程收到回复后，验证是否是自己发起的询问(验证zxid是否一致)，然后获取对方的id(myid)，并存储到当前询问对象列表中，最后获取对方提议的leader相关信息(id,zxid)，并将这些信息存储到当次选举的投票记录表中；

4.收到所有Server回复以后，就计算出zxid最大的那个Server，并将这个Server相关信息设置成**下一次要投票的Server**

5.线程将当前zxid最大的Server设置为当前Server要推荐的Leader，如果此时获胜的Server获得n/2 + 1的Server票数， 设置当前推荐的leader为获胜的Server，将根据获胜的Server相关信息设置自己的状态，否则，继续这个过程，直到leader被选举出来。

通过流程分析我们可以得出：要使Leader获得多数Server的支持，则Server总数必须是奇数2n+1，且存活的Server的数目不得少于n+1。

fast paxos流程是在选举过程中，某Server首先向所有Server提议自己要成为leader，当其它Server收到提议以后，解决epoch和zxid的冲突，并接受对方的提议，然后向对方发送接受提议完成的消息，重复这个流程，最后一定能选举出Leader。

[![F7Yn4P.png](https://s2.ax1x.com/2019/01/05/F7Yn4P.png)](https://s2.ax1x.com/2019/01/05/F7Yn4P.png)

### 原子广播

所有的**写操作**请求被传送给领导者，并通过**广播**将更新信息告诉跟随者。当大部分跟随者执行了修改之后，领导者就提交更新操作，客户端将得到更新成功的回应。未获得一致性的协议被设计为原子的，因此无论修改失败与否，他都分两阶段提交。

[![F7YtNq.png](https://s2.ax1x.com/2019/01/05/F7YtNq.png)](https://s2.ax1x.com/2019/01/05/F7YtNq.png)

如果领导者出故障了，剩下的机器将会再次进行领导者选举，并在新领导被选出前继续执行任务。如果在不久后老的领导者恢复了，那么它将以跟随者的身份继续运行。领导者选举非常快，由发布的结果所知，大约是200毫秒，因此在选举时性能不会明显减慢。

所有在ensemble中的机器在更新他们内存中的Znode树之前会将更新信息写入磁盘。读操作请求可由任何机器服务，他们只设计内存查找，因此非常快。

1. leader等待server连接；
2. Follower连接leader，将最大的zxid发送给leader；
3. Leader根据follower的zxid确定同步点；
4. 完成同步后通知follower 已经成为uptodate状态；
5. Follower收到uptodate消息后，又可以重新接受client的请求进行服务了。

[![F7Yb5t.png](https://s2.ax1x.com/2019/01/05/F7Yb5t.png)](https://s2.ax1x.com/2019/01/05/F7Yb5t.png)

每一个Znode树的更新都会给定一个唯一的全局标识，叫zxid（表示ZooKeeper事务“ID”）。更新是被排序的，因此如果zxid的z1<z2，那么z1就比z2先执行。对于ZooKeeper来说，这是分布式系统中排序的唯一标准。

所有在ensemble中的机器在更新它们内存中的Znode树之前会先**将更新信息写入磁盘**。读操作请求可由任何机器服务，同时，由于他们只涉及内存查找，因此非常快。

### Leader工作流程

Leader主要有三个功能：

1 .**恢复数据**；

2 .维持与Learner的心跳，接收Learner请求并判断Learner的请求消息类型；

3 .Learner的消息类型主要有PING消息、REQUEST消息、ACK消息、REVALIDATE消息，根据不同的消息类型，进行不同的处理。

PING消息是指Learner的心跳信息；

REQUEST消息是Follower发送的提议信息，包括写请求及同步请求；

ACK消息是Follower的对提议的回复，超过半数的Follower通过，则commit该提议；

REVALIDATE消息是用来延长SESSION有效时间。

[![F7tkGV.png](https://s2.ax1x.com/2019/01/05/F7tkGV.png)](https://s2.ax1x.com/2019/01/05/F7tkGV.png)

### Follower工作流程

Follower主要有四个功能：

1. 向Leader发送请求（PING消息、REQUEST消息、ACK消息、REVALIDATE消息）；
2. 接收Leader消息并进行处理；
3. 接收Client的请求，如果为写请求，发送给Leader进行投票；
4. 返回Client结果。

Follower的消息循环处理如下几种来自Leader的消息：

1 .PING消息： 心跳消息；

2 .PROPOSAL消息：Leader发起的提案，要求Follower投票；

3 .COMMIT消息：服务器端最新一次提案的信息；

4 .UPTODATE消息：表明同步完成；

5 .REVALIDATE消息：根据Leader的REVALIDATE结果，关闭待revalidate的session还是允许其接受消息；

6 .SYNC消息：返回SYNC结果到客户端，这个消息最初由客户端发起，用来强制得到最新的更新。

Follower的工作流程简图如下所示，在实际实现中，Follower是通过5个线程来实现功能的。

[![F7tuZ9.png](https://s2.ax1x.com/2019/01/05/F7tuZ9.png)](https://s2.ax1x.com/2019/01/05/F7tuZ9.png)

## ZooKeeper一致性

在ensemble中的领导者和跟随着非常灵活，跟随者通过更新号来滞后领导者11，结果导致了只要大部分而不是所有的ensemble中的元素确认更新，就能被提交了。对于ZooKeeper来说，一个较好的智能模式是将客户端连接到跟着领导者的ZooKeeper服务器上。客户端可能被连接到领导者上，但他不能控制它，而且在如下情况时，甚至可能不知道。参见下图：

[![F708zt.png](https://s2.ax1x.com/2019/01/05/F708zt.png)](https://s2.ax1x.com/2019/01/05/F708zt.png)

每一个Znode树的更新都会给定一个唯一的全局标识，叫zxid（表示ZooKeeper事务“ID”）。更新是被排序的，因此如果zxid的z1<z2，那么z1就比z2先执行。对于ZooKeeper来说，这是分布式系统中排序的唯一标准。

ZooKeeper是一种高性能、可扩展的服务。ZooKeeper的读写速度非常快，并且读的速度要比写快。另外，在进行读操作的时候，ZooKeeper依然能够为旧的数据提供服务。这些都是由ZooKeeper所提供的一致性保证的，它具有如下特点：

（1）顺序一致性

任何一个客户端的更新都按他们发送的顺序排序，也就意味着如果一个客户端将Znode z的值更新为值a，那么在之后的操作中，他会将z更新为b，在客户端发现z带有值b之后，就不会再看见带有值a的z。

（2）原子性

更新不成功就失败，这意味着如果更新失败了，没有客户端会知道。☆☆

（3）单系统映像

无论客户端连接的是哪台服务器，他与系统看见的视图一样。这就意味着，如果一个客户端在相同的会话时连接了一台新的服务器，他将不会再看见比在之前服务器上看见的更老的系统状态，当服务器系统出故障，同时客户端尝试连接ensemble中的其他机器时，故障服务器的后面那台机器将不会接受连接，直到它连接到故障服务器。

（4）容错性

一旦更新成功后，那么在客户端再次更新他之前，他就固定了，将不再被修改，这就会保证产生下面两种结果：

如果客户端成功的获得了正确的返回代码，那么说明更新已经成功。如果不能够获得返回代码（由于通信错误、超时等原因），那么客户端将不知道更新是否生效。

当故障恢复的时候，任何客户端能够看到的执行成功的更新操作将不会回滚。

（5）实时性

在任何客户端的系统视图上的的时间间隔是有限的，因此他在超过几十秒的时间内部会过期。这就意味着，服务器不会让客户端看一些过时的数据，而是关闭，强制客户端转到一个更新的服务器上。

解释一下：

由于性能原因，读操作由ZooKeeper服务器的内存提供，而且不参与写操作的全局排序。这一特性可能会导致来自使用ZooKeeper外部机制交流的客户端与ZooKeeper状态的不一致。举例来说，客户端A将Znode z的值a更新为a’，A让B来读z，B读到z的值是a而不是a’。这与ZooKeeper的保证机制是相容的（不允许的情况较作“同步一致的交叉客户端视 图”）。为了避免这种情况的发生，B在读取z的值之前，应该先调用z上的sync。Sync操作强制B连接上的ZooKeeper服务器与leader保 持一致这样，当B读到z的值时，他将成为A设置的值（或是之后的值）

容易混淆的是：

sync操作只能被异步调用12。这样操作的原因是你不需要等待他的返回，因为ZooKeeper保证了任何接下去的操作将会发生在sync在服务器上执行以后，即使操作是在sync完成前被调用的。

这些已执行的保证后，ZooKeeper更高级功能的设计与实现将会变得非常容易，例如：leader选举、队列，以及可撤销锁等机制的实现。

## 会话

ZooKeeper**客户端**与ensemble中的服务器列表配置一致，在启动时，他尝试与表中的一个服务器相连接。如果连接失败了，他就尝试表中的其他服务器，以此类推，知道他最终连接到其中一个，或者ZooKeeper的所有服务器都无法获得时，连接失败。
一旦与ZooKeeper服务器连接成功，服务器会创建与客户端的一个新的对话。每个回话都有**超时时段**，这是应用程序在创建它时设定的。如果服务器没有在超时时段内得到请求，他可能会中断这个会话。一旦会话被中断了，他可能不再被打开，而且任何与会话相连接的临时节点都将丢失。
无论什么时候会话持续空闲长达一定时间，都会由客户端发送ping请求保持活跃（犹如心跳）。时间段要足够小以监测服务器故障（由读操作超时反应），并且能再回话超市时间段内重新连接到另一个服务器。
在ZooKeeper中有几个time参数。**tick time**是ZooKeeper中的基本时间长度，为ensemble里的服务器所使用，用来定义对于交互运行的调度。其他设置以tick time的名义定义，或者至少由它来约束。
创建更复杂的临时性状态的应用程序应该支持更长的会话超时，因为重新构建的代价会更昂贵。在一些情况下，我们可以让应用程序在一定会话时间内能够重启，并且避免会话过期。（这可能更适合执行维护或是升级）每个会话都由服务器给定一个唯一的身份和密码，而且如果是在建立连接时被传递给ZooKeeper的话，只要没有过期它能够恢复会话。
这些特性可以视为一种可以避免会话过期的优化，但它并不能代替用来处理会话过期。会话过期可能出现在机器突然故障时，或是由于任何原因导致的应用程序安全关闭了，但在会话中断前没有重启。

Zookeeper对象的转变是通过生命周期中的不同状态来实现的。可以使用getState()方法在任何时候查询他的状态：

public states getSate()

Zookeeper状态事务：

[![F70yQ0.png](https://s2.ax1x.com/2019/01/05/F70yQ0.png)](https://s2.ax1x.com/2019/01/05/F70yQ0.png)

getState()方法的返回类型是states，states是枚举类型代表Zookeeper对象可能所处的不同状态，一个Zookeeper实例可能一次只处于一个状态。一个新建的Zookeeper实例正在于Zookeeper服务器建立连接时，是处于**CONNECTING**状态的。一旦连接建立好以后，他就变成了**Connected**状态。
使用Zookeeper的客户端可以通过注册Watcher的方法来获取状态转变的消息。一旦进入了CONNNECTED状态，Watcher将获得一个KeepState值为SyncConnected的WatchedEvent。

注意Zookeeper的状态改变。传递给Zookeeper对象构造函数的(默认)watcher，被用来检测状态的改变。

1. 了解Zookeeper的状态改变。传递给Zookeeper对象构造函数的(默认)watcher，被用来检测状态的改变。
2. 了解Znode的改变。检测Znode的改变既可以使用专门的实例设置到读操说，也可以使用读操作默认的watcher。

Zookeeper实例可能时区或重新连接Zookeeper服务，在CONNETCTED和CONNECTING状态中切换。如果连接段考，watcher得到一个Disconnected事件，需要注意的是这些状态的迁移是由Z哦哦keeper实例自己发起的，如果连接断开他将自动尝试自动连接。

如果任何一个close()方法调用，或者会话由Expired了的KeepState提示过期时,Zookeeper可能会装变成第三种状态CLOSED状态。一旦处于CLOSE状态,Zookeeper的对象将不再是活动的了(可以使用states的isActive()方法进行测试,而且不能被重用。客户端必须建立一个新的Zookeeper实例才能连接到Zookeeper服务。

## 应用场景

Zookeeper是一个高可用的分布式数据管理与系统协调框架。基于Paxos算法实现的，使该框架保证了分布式环境中数据的强一致性，也是基于这样的特性，使得Zookeeper解决了很多分布式问题。

### 数据发布与订阅(配置中心)

　数据发布/订阅系统，即配置中心。需要发布者将数据发布到Zookeeper的节点上，供订阅者进行数据订阅，进而达到动态获取数据的目的，实现配置信息的集中式管理和数据的动态更新。发布/订阅一般有两种设计模式：推模式和拉模式，服务端主动将数据更新发送给所有订阅的客户端称为推模式；客户端主动请求获取最新数据称为拉模式，Zookeeper采用了推拉相结合的模式，客户端向服务端注册自己需要关注的节点，一旦该节点数据发生变更，那么服务端就会向相应的客户端推送Watcher事件通知，客户端接收到此通知后，主动到服务端获取最新的数据。

　　若将配置信息存放到Zookeeper上进行集中管理，在通常情况下，应用在启动时会主动到Zookeeper服务端上进行一次配置信息的获取，同时，在指定节点上注册一个Watcher监听，这样在配置信息发生变更，服务端都会实时通知所有订阅的客户端，从而达到实时获取最新配置的目的。

 分布式搜索服务中，索引的元数据信息和服务器集群机器的节点状态放在ZK的一些指定系欸但，共各个客户端订阅使用。

 分布式日志收集系统。这个系统的核心工作使收集分布在不同机器的日志。收集器通常使按照应用来分配收集任务端元，因此需要在ZK上创建一个以应用名位path的节点P，并将这个应用的所有机器ip，以子节点的形式注册到节点P上，这样一来能够实现机器变动的时候，能够实时通知到收集器调整任务分配。

系统中有些信息需要动态获取，并且还会存在人工手动去修改这个信息的发问。通常是暴露出接口，例如JMX接口，来获取一些运行时的信息。引入ZK之后，就不用自己实现一套方案了，只要将这些信息存放到指定的ZK节点上即可。

[![F7rBJH.png](https://s2.ax1x.com/2019/01/05/F7rBJH.png)](https://s2.ax1x.com/2019/01/05/F7rBJH.png)

### 负载均衡

　　负载均衡是一种相当常见的计算机网络技术，用来对多个计算机、网络连接、CPU、磁盘驱动或其他资源进行分配负载，以达到优化资源使用、最大化吞吐率、最小化响应时间和避免过载的目的。

　　使用Zookeeper实现动态DNS服务

- 域名配置：首先在Zookeeper上创建一个节点来进行域名配置，如DDNS/app1/server.app1.company1.com。
- 域名解析：**应用首先从域名节点中获取IP地址和端口的配置，进行自行解析。同时，应用程序还会在域名节点上注册一个数据变更Watcher监听，以便及时收到域名变更的通知。**
- 域名变更：若发生IP或端口号变更，此时需要进行域名变更操作，此时，只需要对指定的域名节点进行更新操作，Zookeeper就会向订阅的客户端发送这个事件通知，客户端之后就再次进行域名配置的获取。

### 命名服务

　命名服务是分步实现系统中较为常见的一类场景，分布式系统中，被命名的实体通常可以是集群中的机器、提供的服务地址或远程对象等，通过命名服务，客户端可以根据指定名字来获取资源的实体、服务地址和提供者的信息。Zookeeper也可帮助应用系统通过资源引用的方式来实现对资源的定位和使用，广义上的命名服务的资源定位都不是真正意义上的实体资源，在分布式环境中，上层应用仅仅需要一个全局唯一的名字。Zookeeper可以实现一套分布式全局唯一ID的分配机制。

阿里巴巴集团开源的分布式服务框架Dubbo中使用Zookeeper为命名服务，维护全局的地址服务列表在Dubbo实现中：

服务提供自己的提供者在启动的时候，向ZK上指定节点/dubbo/${serviceName}/providers目录下写入自己的URL地址，这个操作就完成了服务的发布。

服务消费者启动的时候订阅/dubbo/serviceName/providers目录下提供者URL地址，并向/dubbo/serviceName/providers目录下提供者URL地址，并向/dubbo/serviceName}/consumers写下自己的URL地址。

**注意：所有向ZK上注册的地址都是临时节点，这样就能够保证服务提供者和消费者能过够自动感应资源的拜年话。另外，Dubbo还针对服务粒度的监控，方法是订阅/dubbo/${serviceName}目录下所有提供者和消费者信息**

[![F7skTO.png](https://s2.ax1x.com/2019/01/05/F7skTO.png)](https://s2.ax1x.com/2019/01/05/F7skTO.png)

### 分布式通知/协调

　　Zookeeper中特有的Watcher注册于异步通知机制，能够很好地实现分布式环境下不同机器，甚至不同系统之间的协调与通知，从而实现对数据变更的实时处理。通常的做法是不同的客户端都对Zookeeper上的同一个数据节点进行Watcher注册，监听数据节点的变化（包括节点本身和子节点），若数据节点发生变化，那么所有订阅的客户端都能够接收到相应的Watcher通知，并作出相应处理。

　在绝大多数分布式系统中，系统机器间的通信无外乎**心跳检测**、**工作进度汇报**和**系统调度**。

　　① **心跳检测**，不同机器间需要检测到彼此是否在正常运行，可以使用Zookeeper实现机器间的心跳检测，基于其临时节点特性（临时节点的生存周期是客户端会话，客户端若当即后，其临时节点自然不再存在），可以让不同机器都在Zookeeper的一个指定节点下创建临时子节点，不同的机器之间可以根据这个临时子节点来判断对应的客户端机器是否存活。通过Zookeeper可以大大减少系统耦合。

　　② **工作进度汇报**，通常任务被分发到不同机器后，需要实时地将自己的任务执行进度汇报给分发系统，可以在Zookeeper上选择一个节点，每个任务客户端都在这个节点下面创建临时子节点，这样不仅可以判断机器是否存活，同时各个机器可以将自己的任务执行进度写到该临时节点中去，以便中心系统能够实时获取任务的执行进度。

　　③ **系统调度**，Zookeeper能够实现如下系统调度模式：分布式系统由控制台和一些客户端系统两部分构成，控制台的职责就是需要将一些指令信息发送给所有的客户端，以控制他们进行相应的业务逻辑，后台管理人员在控制台上做一些操作，实际上就是修改Zookeeper上某些节点的数据，Zookeeper可以把数据变更以时间通知的形式发送给订阅客户端。

### 集群管理

　　Zookeeper的两大特性：

　　**· 客户端如果对Zookeeper的数据节点注册Watcher监听，那么当该数据及诶单内容或是其子节点列表发生变更时，Zookeeper服务器就会向订阅的客户端发送变更通知。**

　　**· 对在Zookeeper上创建的临时节点，一旦客户端与服务器之间的会话失效，那么临时节点也会被自动删除。**

　　利用其两大特性，可以实现集群机器存活监控系统，若监控系统在/clusterServers节点上注册一个Watcher监听，那么但凡进行动态添加机器的操作，就会在/clusterServers节点下创建一个临时节点：/clusterServers/[Hostname]，这样，监控系统就能够实时监测机器的变动情况。下面通过分布式日志收集系统的典型应用来学习Zookeeper如何实现集群管理。

　　分布式日志收集系统的核心工作就是收集分布在不同机器上的系统日志，在典型的日志系统架构设计中，整个日志系统会把所有需要收集的日志机器分为多个组别，每个组别对应一个收集器，这个收集器其实就是一个后台机器，用于收集日志，对于大规模的分布式日志收集系统场景，通常需要解决两个问题：

　　**· 变化的日志源机器**

　　**· 变化的收集器机器**

　　无论是日志源机器还是收集器机器的变更，最终都可以归结为如何快速、合理、动态地为每个收集器分配对应的日志源机器。使用Zookeeper的场景步骤如下

　　① **注册收集器机器**，在Zookeeper上创建一个节点作为收集器的根节点，例如/logs/collector的收集器节点，每个收集器机器启动时都会在收集器节点下创建自己的节点，如/logs/collector/[Hostname]

[![img](https://images2015.cnblogs.com/blog/616953/201611/616953-20161112085605967-1265109617.png)](https://images2015.cnblogs.com/blog/616953/201611/616953-20161112085605967-1265109617.png)

　　② **任务分发**，所有收集器机器都创建完对应节点后，系统根据收集器节点下子节点的个数，将所有日志源机器分成对应的若干组，然后将分组后的机器列表分别写到这些收集器机器创建的子节点，如/logs/collector/host1上去。这样，收集器机器就能够根据自己对应的收集器节点上获取日志源机器列表，进而开始进行日志收集工作。

　　③ **状态汇报**，完成任务分发后，机器随时会宕机，所以需要有一个收集器的状态汇报机制，每个收集器机器上创建完节点后，还需要再对应子节点上创建一个状态子节点，如/logs/collector/host/status，每个收集器机器都需要定期向该结点写入自己的状态信息，这可看做是心跳检测机制，通常收集器机器都会写入日志收集状态信息，日志系统通过判断状态子节点最后的更新时间来确定收集器机器是否存活。

　　④ **动态分配**，若收集器机器宕机，则需要动态进行收集任务的分配，收集系统运行过程中关注/logs/collector节点下所有子节点的变更，一旦有机器停止汇报或有新机器加入，就开始进行任务的重新分配，此时通常由两种做法：

　　**· 全局动态分配**，当收集器机器宕机或有新的机器加入，系统根据新的收集器机器列表，立即对所有的日志源机器重新进行一次分组，然后将其分配给剩下的收集器机器。

　　**· 局部动态分配**，每个收集器机器在汇报自己日志收集状态的同时，也会把自己的负载汇报上去，如果一个机器宕机了，那么日志系统就会把之前分配给这个机器的任务重新分配到那些负载较低的机器，同样，如果有新机器加入，会从那些负载高的机器上转移一部分任务给新机器。

　　上述步骤已经完整的说明了整个日志收集系统的工作流程，其中有两点注意事项。

　　①**节点类型**，在/logs/collector节点下创建临时节点可以很好的判断机器是否存活，但是，若机器挂了，其节点会被删除，记录在节点上的日志源机器列表也被清除，所以需要选择持久节点来标识每一台机器，同时在节点下分别创建/logs/collector/[Hostname]/status节点来表征每一个收集器机器的状态，这样，既能实现对所有机器的监控，同时机器挂掉后，依然能够将分配任务还原。

　　② **日志系统节点监听**，若采用Watcher机制，那么通知的消息量的网络开销非常大，需要采用日志系统主动轮询收集器节点的策略，这样可以节省网络流量，但是存在一定的延时。

　　2.6 Master选举

　　在分布式系统中，Master往往用来协调集群中其他系统单元，具有对分布式系统状态变更的决定权，如在读写分离的应用场景中，客户端的写请求往往是由Master来处理，或者其常常处理一些复杂的逻辑并将处理结果同步给其他系统单元。利用Zookeeper的强一致性，能够很好地保证在分布式高并发情况下节点的创建一定能够保证全局唯一性，即Zookeeper将会保证客户端无法重复创建一个已经存在的数据节点。

　　首先创建/master_election/2016-11-12节点，客户端集群每天会定时往该节点下创建临时节点，如/master_election/2016-11-12/binding，这个过程中，只有一个客户端能够成功创建，此时其变成master，其他节点都会在节点/master_election/2016-11-12上注册一个子节点变更的Watcher，用于监控当前的Master机器是否存活，一旦发现当前Master挂了，其余客户端将会重新进行Master选举。

[![img](https://images2015.cnblogs.com/blog/616953/201611/616953-20161112093212483-1690012026.png)](https://images2015.cnblogs.com/blog/616953/201611/616953-20161112093212483-1690012026.png)

### 分布式锁

在Zookeeper，分布式锁使全部同步的，在同一时刻不会由相同的客户客户端认为他们持有相同的锁：

1. Zookeeper调用create()方法来床架哪一个路径格式位“_locknode_/lock”的节点，此节点类型为sequence(连续)和ephemeral(临时)。创建的节点位临时节点，并且所有的节点连续编号，即为“lock-i”的格式。
2. 在创建锁节点上调用getChildren()方法，以获取锁目录下最小编号节点，并且不设置watch；
3. 步骤2获取的节点桥好事步骤1客户端创建的节点，客户端会获取该种类型的锁，然后退出操作。
4. 客户端在锁目录上调用exists()方法，并且设置watch来监视锁目录下序号对应自己次小的连续临时节点的状态。
5. 如果监视节点状态发生变化，则跳转到步骤2，继续进行后续的操作，直至退出锁竞争。

Zookeeper的解锁非常简单，客户端只需要将加索步骤1中创建的临时节点和删除即可。

[![F76kJe.png](https://s2.ax1x.com/2019/01/05/F76kJe.png)](https://s2.ax1x.com/2019/01/05/F76kJe.png)

## 参考博客

<https://www.cnblogs.com/leesf456/p/6036548.html>

<https://blog.csdn.net/kingcat666/article/details/77749547>