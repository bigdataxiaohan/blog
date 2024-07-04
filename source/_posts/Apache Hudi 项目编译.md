---
title: Apache Hudi项目编译
date: 2022-08-09 21:36:01
tags: Hudi
categories: Hudi
---

### 前置环境

| 环境  | 版本      |
| ----- | --------- |
| java  | 1.8.0_271 |
| maven | 3.8.5     |
| scala | 2.12      |

### 拉取Hudi源码

```shell
git clone https://github.com/apache/hudi.git
```

使用IDEA打开hudi项目将分支切换到 `release-0.12.0`

修改hudi项目的pom.xml文件添加加速依赖

```xml
<repository>
        <id>nexus-aliyun</id>
        <name>nexus-aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
```

修改依赖的组件版本

```xml
<hadoop.version>3.1.3</hadoop.version>
<hive.version>3.1.2</hive.version>
```

打包命令

```shell
mvn clean package -DskipTests -Dspark3.2 -Dflink1.13 -Dscala-2.12 -Dhadoop.version=3.1.3 -Pflink-bundle-shade-hive3
```

![image-20221031230126177](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221031230126177.png)

### 修改源码适配Hadoop3.X

Hudi默认依赖的hadoop2，要兼容hadoop3，除了修改版本，HoodieParquetDataBlock，这里需要传入两个参数因此我们将地119行修改为

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/数据湖/20221025213122.png)

```java
    try (FSDataOutputStream outputStream = new FSDataOutputStream(baos,null)) {
      try (HoodieParquetStreamWriter<IndexedRecord> parquetWriter = new HoodieParquetStreamWriter<>(outputStream, avroParquetConfig)) {
        for (IndexedRecord record : records) {
          String recordKey = getRecordKey(record).orElse(null);
          parquetWriter.writeAvro(recordKey, record);
        }
        outputStream.flush();
      }
    }
```



### 解决Jetty冲突

![image-20221108091948204](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221108091948204.png)

原因：Hive版本为3.1.2，其携带的jetty是0.9.3，hudi本身用的0.9.4，存在依赖冲突。

修改` hudi-0.12.0/packaging/hudi-spark-bundle/pom.xml`  在382行的位置，修改如下

```xml
<!-- Hive -->
    <dependency>
      <groupId>${hive.groupid}</groupId>
      <artifactId>hive-service</artifactId>
      <version>${hive.version}</version>
      <scope>${spark.bundle.hive.scope}</scope>
      <exclusions>
        <exclusion>
          <artifactId>guava</artifactId>
          <groupId>com.google.guava</groupId>
        </exclusion>
        <exclusion>
          <groupId>org.eclipse.jetty</groupId>
          <artifactId>*</artifactId>
        </exclusion>
        <exclusion>
          <groupId>org.pentaho</groupId>
          <artifactId>*</artifactId>
        </exclusion>
      </exclusions>
    </dependency>

    <dependency>
      <groupId>${hive.groupid}</groupId>
      <artifactId>hive-service-rpc</artifactId>
      <version>${hive.version}</version>
      <scope>${spark.bundle.hive.scope}</scope>
    </dependency>

    <dependency>
      <groupId>${hive.groupid}</groupId>
      <artifactId>hive-jdbc</artifactId>
      <version>${hive.version}</version>
      <scope>${spark.bundle.hive.scope}</scope>
      <exclusions>
        <exclusion>
          <groupId>javax.servlet</groupId>
          <artifactId>*</artifactId>
        </exclusion>
        <exclusion>
          <groupId>javax.servlet.jsp</groupId>
          <artifactId>*</artifactId>
        </exclusion>
        <exclusion>
          <groupId>org.eclipse.jetty</groupId>
          <artifactId>*</artifactId>
        </exclusion>
      </exclusions>
    </dependency>

    <dependency>
      <groupId>${hive.groupid}</groupId>
      <artifactId>hive-metastore</artifactId>
      <version>${hive.version}</version>
      <scope>${spark.bundle.hive.scope}</scope>
      <exclusions>
        <exclusion>
          <groupId>javax.servlet</groupId>
          <artifactId>*</artifactId>
        </exclusion>
        <exclusion>
          <groupId>org.datanucleus</groupId>
          <artifactId>datanucleus-core</artifactId>
        </exclusion>
        <exclusion>
          <groupId>javax.servlet.jsp</groupId>
          <artifactId>*</artifactId>
        </exclusion>
        <exclusion>
          <artifactId>guava</artifactId>
          <groupId>com.google.guava</groupId>
        </exclusion>
      </exclusions>
    </dependency>

    <dependency>
      <groupId>${hive.groupid}</groupId>
      <artifactId>hive-common</artifactId>
      <version>${hive.version}</version>
      <scope>${spark.bundle.hive.scope}</scope>
      <exclusions>
        <exclusion>
          <groupId>org.eclipse.jetty.orbit</groupId>
          <artifactId>javax.servlet</artifactId>
        </exclusion>
        <exclusion>
          <groupId>org.eclipse.jetty</groupId>
          <artifactId>*</artifactId>
        </exclusion>
      </exclusions>
</dependency>

    <!-- 增加hudi配置版本的jetty -->
    <dependency>
      <groupId>org.eclipse.jetty</groupId>
      <artifactId>jetty-server</artifactId>
      <version>${jetty.version}</version>
    </dependency>
    <dependency>
      <groupId>org.eclipse.jetty</groupId>
      <artifactId>jetty-util</artifactId>
      <version>${jetty.version}</version>
    </dependency>
    <dependency>
      <groupId>org.eclipse.jetty</groupId>
      <artifactId>jetty-webapp</artifactId>
      <version>${jetty.version}</version>
    </dependency>
    <dependency>
      <groupId>org.eclipse.jetty</groupId>
      <artifactId>jetty-http</artifactId>
      <version>${jetty.version}</version>
    </dependency>
```





