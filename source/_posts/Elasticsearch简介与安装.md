---
title: Elasticsearch简介与安装
date: 2018-12-15 10:41:09
mathjax: true
tags: Elasticsearch
categories: NOSQL
---

## 搜索

就是在任何场景下，找寻你想要的信息，这个时候，会输入一段你要搜索的关键字，然后就期望找到这个关键字相关的有些信息

### 垂直搜索

站内搜索

### 互联网搜索

电商网站，招聘网站，新闻网站，各种app

### IT系统的搜索

OA软件，办公自动化软件，会议管理，日程管理，项目管理，员工管理，搜索“张三”，“张三儿”，“张小三”；有个电商网站，卖家，后台管理系统，搜索“牙膏”，订单，“牙膏相关的订单”

数据都是存储在数据库里面的，比如说电商网站的商品信息，招聘网站的职位信息，新闻网站的新闻信息，如果说从技术的角度去考虑，如何实现如说，电商网站内部的搜索功能的话，就可以考虑，去使用数据库去进行搜索。

1、每条记录的指定字段的文本，可能会很长，比如说“商品描述”字段的长度，有长达数千个，甚至数万个字符，这个时候，每次都要对每条记录的所有文本进行扫描，来判断说，你包不包含我指定的这个关键词（比如说“牙膏”）
2、还不能将搜索词拆分开来，尽可能去搜索更多的符合你的期望的结果，比如输入“生化机”，就搜索不出来“生化危机”

用数据库来实现搜索，是不太靠谱的。通常来说，性能会很差的。

## Lucene

Lucene是Apache软件基金会中一个开放源代码的全文搜索引擎工具包，是一个全文搜索引擎的架构，提供了完整的查询引擎和索引引擎，部分文本分析引擎。Lucene的目的是为软件开发人员提供一个简单易用的工具包，以方便在目标系统中实现全文检索的功能，或者是以此为基础建立起完整的全文搜索引擎。

[全文检索](https://baike.baidu.com/item/%E5%85%A8%E6%96%87%E6%A3%80%E7%B4%A2/8028630)是指计算机索引程序通过扫描文章中的每一个词，对每一个词建立一个索引，指明该词在文章中出现的次数和位置，当用户查询时，检索程序就根据事先建立的索引进行查找，并将查找的结果反馈给用户的检索方式。这个过程类似于通过字典中的检索字表查字的过程。全文搜索搜索引擎数据库中的数据。

倒排索引源于实际应用中需要根据属性的值来查找记录。这种索引表中的每一项都包括一个属性值和具有该属性值的各记录的地址由于不是由记录来确定属性值，而是由属性值来确定记录的位置，因而称为倒排索引（inverted index)。带有倒排索引的文件我们称为倒排索引文件，简称倒排文件（inverted file)。倒排索引中的索引对象是文档或者文档集合中的单词等，用来存储这些单词在一个文档或者一组文档中的存储位置，是对文档或者文档集合的一种最常用的索引机制。搜索引擎的关键步骤就是建立倒排索引，倒排索引一般表示为一个关键词，然后是它的频度（岀现的次数)、位置（出现在哪一篇文章或网页中，及有关的日期，作者等信息)，好比一本书的目录、标签一般。读者想看哪一个主题相关的章节，直接根据目录即可找到相关的页面。不必再从书的第一页到最后一页，一页一页地查找。

###  评分公式

文档得分:它是一个刻画文档与査询匹配程度的参数。Apache Lucene的默认评分机制：TF/IDF (词频/逆文档频率）算法

当一个文档经Lucene返回，则意味着该文档与用户提交的查询是匹配的。在这种情况下，每个返回的文档都有一个得分。得分越高，文档相关度更高，同一个文档针对不同查询的得分是不同的，比较某文档在不同查询中的得分是没有意义的。同一文档在不同查询中的得分不具备可比较性，不同查询返回文档中的最高得分也不具备可比较性。这是因为文档得分依赖多个因子，除了权重和查询本身的结构，还包括匹配的词项数目，词项所在字段，以及用于查询规范化的匹配类型等。在一些比较极端的情况下，同一个文档在相似查询中的得分非常悬殊，仅仅是因为使用了自定义得分查询或者命中词项数发生了急剧变化。

#### 参考因子

- 文档权重（document boost):索引期赋予某个文档的权重值。
- 字段权重（field boost):查询期赋予某个字段的权重值。
- 协调因子（coord):基于文档中词项命中个数的协调因子，一个文档命中了查询中的词项越多，得分越高。
- 逆文档频率（inverse document frequency): —个基于词项的因子，用来告诉评分公式该词项有多么罕见。逆文档频率越低，词项越罕见。评分公式利用该因子为包含罕见词项的文档加权。
- 长度范数（length nomi):每个字段的基于词项个数的归一化因子（在索引期计算出来并存储在索引中）。一个字段包含的词项数越多，该因子的权重越低，这意味着 Apache Lucene评分公式更“喜欢”包含更少词项的字段。
- 词频（term frequency): —个基于词项的因子，用来表示一个词项在某个文档中出现了多少次。词频越高，文档得分越高。
- 查询范数（query norm): —个基于查询的归一化因子，它等于查询中词项的权重平方和。查询范数使不同查询的得分能相互比较，尽管这种比较通常是困难且不可行的。

#### TF/IDF评分公式

$$score(q,d)=coord(q,d)*queryBoost(q)*\frac{V(q)* {V(d)}}{|V(q)|}lengthNorm(d)*docBoost(d)$$

上面公式糅合了布尔检索模型和向量空间检索模型。想了解更多请百度。

#### 实际公式

$$score(q,d)=coord(q,d)*queryBoost(q)*\sum_{i \ \ in \ \ q}(tf ( t \ \  in \ \ d) *idf(t)^2*boost(t)*norm(t,d))$$

得分公式是一个关于査询q和文档d的函数，有两个因子coord和queryNorm并不直接依赖查询词项，而是与查询词项的一个求和公式相乘。求和公式中每个加数由以下因子连乘所得：词频、逆文档频率、词项权重、范数。范数就是之前提到的长度范数。

基本规则：

- 越多罕见的词项被匹配上，文档得分越高。
- 文档字段越短（包含更少的词项)，文档得分越高。
- 权重越高（不论是索引期还是査询期赋予的权重值)，文档得分越高。

## Elasticsearch

Elasticsearch (ES)是一个基于Lucene构建的开源、分布式、[RESTful](https://baike.baidu.com/item/RESTful/4406165?fr=aladdin)接口全文搜索引擎Elasticsearch还是一个分布式文档数据库，其中每个字段均是被索引的数据且可被搜索，它能够扩展至数以百计的服务器存储以及处理PB级的数据。它可以在很短的时间内存储、搜索和分析大量的数据。它通常作为具有复杂搜索场景情况下的核心发动机。Elasticsearch就是为高可用和可扩展而生的。可以通过购置性能更强的服务器来完成，称为垂直扩展或者向上扩展（Vertical Scale/Scaling Up)，或增加更多的服务器来完成，称为水平扩展或者向外扩展（Horizontal Scale/Scaling Out)尽管ES能够利用更强劲的硬件，垂直扩展毕竟还是有它的极限。真正的可扩展性来自于水平扩展，通过向集群中添加更多的节点来分担负载，增加可靠性。在大多数数据库中，水平扩展通常都需要你对应用进行一次大的重构来利用更多的节点。而ES天生就是分布式的：它知道如何管理多个节点来完成扩展和实现高可用性。这也意味着你的应用不需要做任何的改动。

### 评分规则

是ElasticSearch使用了Lucene的评分功能，但好在我们可以替换默认的评分算法，ElasticSearch使用了Lucene的评分功
能但不仅限于Lucene的评分功能。用户可以使用各种不同的查询类型以精确控制文档评分的计算（custom_boost_factor 查询、constant_score 査询、custom_score 查询等），还可以通过使用脚本（scripting)来改变文档得分，还可以使用ElasticSearch 0.90中岀现的二次评分功能，通过在返回文档集之上执行另外一个查询，重新计算前N个文档的文档得分。

- Okapi BM25模型：这是一种基于概率模型的相似度模型，可用于估算文档与给定査询匹配的概率。为了在ElasticSearch中使用它，你需要使用该模型的名字，BM25。一般来说，Okapi BM25模型在短文本文档上的效果最好，因为这种场景中重复词项对文档的总体得分损害较大。
- 随机偏离（Divergence from randomness)模型：这是一种基于同名概率模型的相似度模型。为了在ElasticSearch中使用它，你需要使用该模型的名字，DFR。一般来说，随机偏离模型在类似自然语言的文本上效果较好。
- 基于信息的（Information based)模型：这是最后一个新引人的相似度模型，与随机偏离模型类似。为了在ElasticSearch中使用它，你需要使用该模型的名字，IB。同样，IB模型也在类似自然语言的文本上拥有较好的效果。

### 用途

- 分布式实时文件存储，并将每一个字段都编入索引，使其可以被搜索。
- 实时分析的分布式搜索引擎。
- 可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据。

### 应用场景

#### 国外

- 维基百科，类似百度百科，牙膏，牙膏的维基百科，全文检索，高亮，搜索推荐
- The Guardian（国外新闻网站），类似搜狐新闻，用户行为日志（点击，浏览，收藏，评论）+社交网络数据（对某某新闻的相关看法），数据分析，给到每篇新闻文章的作者，让他知道他的文章的公众反馈（好，坏，热门，垃圾，鄙视，崇拜）
- Stack Overflow（国外的程序异常讨论论坛），IT问题，程序的报错，提交上去，有人会跟你讨论和回答，全文检索，搜索相关问题和答案，程序报错了，就会将报错信息粘贴到里面去，搜索有没有对应的答案
- GitHub（开源代码管理），搜索上千亿行代码
- 电商网站，检索商品
- 日志数据分析，logstash采集日志，ES进行复杂的数据分析（ELK技术，elasticsearch+logstash+kibana）
- 商品价格监控网站，用户设定某商品的价格阈值，当低于该阈值的时候，发送通知消息给用户，比如说订阅牙膏的监控，如果高露洁牙膏的家庭套装低于50块钱，就通知我，我就去买
- BI系统，商业智能，Business Intelligence。比如说有个大型商场集团，BI，分析一下某某区域最近3年的用户消费金额的趋势以及用户群体的组成构成，产出相关的数张报表，**区，最近3年，每年消费金额呈现100%的增长，而且用户群体85%是高级白领，开一个新商场。ES执行数据分析和挖掘，Kibana进行数据可视化

#### 国内

- 国内：站内搜索（电商，招聘，门户，等等），IT系统搜索（OA，CRM，ERP，等等），数据分析（ES热门的一个使用场景）

###  特点

1. 可以作为一个大型分布式集群（数百台服务器）技术，处理PB级数据，服务大公司；也可以运行在单机上，服务小公司
2. Elasticsearch不是什么新技术，主要是将全文检索、数据分析以及分布式技术，合并在了一起，才形成了独一无二的ES；lucene（全文检索），商用的数据分析软件（BI），分布式数据库（mycat）
3. 对用户而言，是开箱即用的，非常简单，作为中小型的应用，直接3分钟部署一下ES，就可以作为生产环境的系统来使用了，数据量不大，操作不是太复杂
4. 数据库的功能面对很多领域是不够用的（事务，还有各种联机事务型的操作）；特殊的功能，比如全文检索，同义词处理，相关度排名，复杂数据分析，海量数据的近实时处理；Elasticsearch作为传统数据库的一个补充，提供了数据库所不不能提供的很多功能

### 核心概念

（1）Near Realtime（NRT）：近实时，两个意思，从写入数据到数据可以被搜索到有一个小延迟（大概1秒）；基于es执行搜索和分析可以达到秒级

（2）Cluster：集群，包含多个节点，每个节点属于哪个集群是通过一个配置（集群名称，默认是elasticsearch）来决定的，对于中小型应用来说，刚开始一个集群就一个节点很正常
（3）Node：节点，集群中的一个节点，节点也有一个名称（默认是随机分配的），节点名称很重要（在执行运维管理操作的时候），默认节点会去加入一个名称为“elasticsearch”的集群，如果直接启动一堆节点，那么它们会自动组成一个elasticsearch集群，当然一个节点也可以组成一个elasticsearch集群

（4）Document&field：文档，es中的最小数据单元，一个document可以是一条客户数据，一条商品分类数据，一条订单数据，通常用JSON数据结构表示，每个index下的type中，都可以去存储多个document。一个document里面有多个field，每个field就是一个数据字段。

```js
#product document
{
  "product_id": "1",
  "product_name": "高露洁牙膏",
  "product_desc": "高效美白",
  "category_id": "2",
  "category_name": "日化用品"
}
```



（5）Index：索引，包含一堆有相似结构的文档数据，比如可以有一个客户索引，商品分类索引，订单索引，索引有一个名称。一个index包含很多document，一个index就代表了一类类似的或者相同的document。比如说建立一个product index，商品索引，里面可能就存放了所有的商品数据，所有的商品document。
（6）Type：类型，每个索引里都可以有一个或多个type，type是index中的一个逻辑数据分类，一个type下的document，都有相同的field，比如博客系统，有一个索引，可以定义用户数据type，博客数据type，评论数据type。

商品index，里面存放了所有的商品数据，商品document

但是商品分很多种类，每个种类的document的field可能不太一样，比如说电器商品，可能还包含一些诸如售后时间范围这样的特殊field；生鲜商品，还包含一些诸如生鲜保质期之类的特殊field

type，日化商品type，电器商品type，生鲜商品type

日化商品type：product_id，product_name，product_desc，category_id，category_name
电器商品type：product_id，product_name，product_desc，category_id，category_name，service_period
生鲜商品type：product_id，product_name，product_desc，category_id，category_name，eat_period

```json
#每一个type里面，都会包含一堆document
{
  "product_id": "2",
  "product_name": "长虹电视机",
  "product_desc": "4k高清",
  "category_id": "3",
  "category_name": "电器",
  "service_period": "1年"
}

{
  "product_id": "3",
  "product_name": "基围虾",
  "product_desc": "纯天然，冰岛产",
  "category_id": "4",
  "category_name": "生鲜",
  "eat_period": "7天"
}
```



（7）shard：单台机器无法存储大量数据，es可以将一个索引中的数据切分为多个shard，分布在多台服务器上存储。有了shard就可以横向扩展，存储更多数据，让搜索和分析等操作分布到多台服务器上去执行，提升吞吐量和性能。每个shard都是一个lucene index。
（8）replica：任何一个服务器随时可能故障或宕机，此时shard可能就会丢失，因此可以为每个shard创建多个replica副本。replica可以在shard故障时提供备用服务，保证数据不丢失，多个replica还可以提升搜索操作的吞吐量和性能。primary shard（建立索引时一次设置，不能修改，默认5个），replica shard（随时修改数量，默认1个），默认每个索引10个shard，5个primary shard，5个replica shard，最小的高可用配置，是2台服务器。

### 对比

| 关系型数据库（比如Mysql） | 非关系型数据库（Elasticsearch） |
| ------------------------- | ------------------------------- |
| 数据库Database            | 索引Index                       |
| 表Table                   | 类型Type                        |
| 数据行Row                 | 文档Document                    |
| 数据列Column              | 字段Field                       |
| 约束 Schema               | 映射Mapping                     |

| HTTP方法 | 数据处理 | 说明                                               |
| -------- | -------- | -------------------------------------------------- |
| POST     | Create   | 新增一个没有ID的资源                               |
| GET      | Read     | 取得一个资源                                       |
| PUT      | Update   | 更新一个资源。或新增一个含ID的资源（如果1D不存在） |
| DELETE   | Delete   | 删除一个资源                                       |



## ES安装

Elasticsearch官网： <https://www.elastic.co/products/elasticsearch>

1）解压elasticsearch-5.2.2.tar.gz到/opt/module目录下

```shell
 tar -zxvf elasticsearch-5.2.2.tar.gz -C /opt/module/
```



2）在/opt/module/elasticsearch-5.2.2路径下创建data和logs文件夹

```shell
mkdir data
mkdir logs
```



3）修改配置文件/opt/module/elasticsearch-5.2.2/config/elasticsearch.yml

```shell
# ---------------------------------- Cluster -----------------------------------
cluster.name: my-application
# ------------------------------------ Node ------------------------------------
node.name: node-102
# ----------------------------------- Paths ------------------------------------
path.data: /opt/module/elasticsearch-5.2.2/data
path.logs: /opt/module/elasticsearch-5.2.2/logs
# ----------------------------------- Memory -----------------------------------
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
# ---------------------------------- Network -----------------------------------
network.host: 192.168.1.11   #自己ip
# --------------------------------- Discovery ----------------------------------
discovery.zen.ping.unicast.hosts: ["elasticsearch"] #自己主机名

#注意
（1）cluster.name如果要配置集群需要两个节点上的elasticsearch配置的cluster.name相同，都启动可以自动组成集群，这里如果不改cluster.name则默认是cluster.name=my-application，
（2）nodename随意取但是集群内的各节点不能相同
（3）修改后的每行前面不能有空格，修改后的“：”后面必须有一个空格
```

4)配置Linux系统

切换到root用户，

```shell
#编辑/etc/security/limits.conf添加类似如下内容
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```

```
 #编辑 /etc/security/limits.d/90-nproc.conf
 * soft nproc 1024
#修改为
* soft nproc 4096
```

```shell
#编辑 vi /etc/sysctl.conf 
vm.max_map_count=655360
fs.file-max=655360
```

```shell
sysctl -p
重新启动elasticsearch，即可启动成功。
```

```shell
#测试集群
curl http://elasticsearch:9200
{
  "name" : "node-1",
  "cluster_name" : "my-application",
  "cluster_uuid" : "mdLmu1rOS7qPNfToNOXzKA",
  "version" : {
    "number" : "5.2.2",
    "build_hash" : "f9d9b74",
    "build_date" : "2017-02-24T17:26:45.835Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.1"
  },
  "tagline" : "You Know, for Search"
}

```

## 插件安装

### 下载插件

<https://github.com/mobz/elasticsearch-head>                  elasticsearch-head-master.zip

<https://nodejs.org/dist/>            node-v6.9.2-linux-x64.tar.xz

### 安装环境

```shell
tar -zxvf node-v6.9.2-linux-x64.tar.gz -C     /opt/module/
vi /etc/profile
export NODE_HOME=/opt/module/node-v6.9.2-linux-x64
export PATH=$PATH:$NODE_HOME/bin
source /etc/profile 
```

### node和npm

```shell
node -v
v6.9.2 
npm -v
3.10.9 
```

### 插件配置

```shell
unzip elasticsearch-head-master.zip -d /opt/module/
```

### 换源

```shell
npm config set registry https://registry.npm.taobao.org
npm config list / npm config get registery  #检查是否替换成功
```

### 安装插件

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 安装grunt:

```shell
npm install -g grunt-cli
```

### 编辑Gruntfile.js

```shell
#文件93行添加hostname:'0.0.0.0'
options: {
        hostname:'0.0.0.0',
        port: 9100,
        base: '.',
        keepalive: true
      }
```

### 检查

```shell
#检查head根目录下是否存在base文件夹
#没有：将 _site下的base文件夹及其内容复制到head根目录下
mkdir base
cp base/* ../base/
```

### 启动grunt server：

```shell
grunt server -d
#如果提示grunt的模块没有安装：
Local Npm module “grunt-contrib-clean” not found. Is it installed? 
Local Npm module “grunt-contrib-concat” not found. Is it installed? 
Local Npm module “grunt-contrib-watch” not found. Is it installed? 
Local Npm module “grunt-contrib-connect” not found. Is it installed? 
Local Npm module “grunt-contrib-copy” not found. Is it installed? 
Local Npm module “grunt-contrib-jasmine” not found. Is it installed? 
Warning: Task “connect:server” not found. Use –force to continue. 

#执行以下命令： 
npm install grunt-contrib-clean -registry=https://registry.npm.taobao.org
npm install grunt-contrib-concat -registry=https://registry.npm.taobao.org
npm install grunt-contrib-watch -registry=https://registry.npm.taobao.org 
npm install grunt-contrib-connect -registry=https://registry.npm.taobao.org
npm install grunt-contrib-copy -registry=https://registry.npm.taobao.org 
npm install grunt-contrib-jasmine -registry=https://registry.npm.taobao.org
#最后一个模块可能安装不成功，但是不影响使用。
```

###  Web访问

![FULpvj.md.png](https://s1.ax1x.com/2018/12/15/FULpvj.md.png)

### 集群无法访问

```shell
在/opt/module/elasticsearch-5.2.2/config路径下修改配置文件elasticsearch.yml，在文件末尾增加
http.cors.enabled: true
http.cors.allow-origin: "*"
#重启ElasticSearch
#重启插件
```

