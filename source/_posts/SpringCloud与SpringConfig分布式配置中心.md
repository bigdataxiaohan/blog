---
title: SpringCloud与SpringConfig分布式配置中心
date: 2019-04-27 19:27:30
categories: 微服务
tags:
- SpringCloud
---

问题

 微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。SpringCloud提供了ConfigServer来解决这个问题，我们每一个微服务自己带着一个application.yml，上百个配置文件管理起来势必使以减十分麻烦的事情因此我们引入了SpringCloud Config。

## 简介

SpringConfig图解

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427193726.png)

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。

SpringCloud Config分为服务端和客户端两部分。

服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口

客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。

## 作用

1. 集中管理配置文件

2. 不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release

3. 运行期间动态调整配置，不需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息

4. 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置

5. 将配置信息以REST接口的形式暴露

## GitHub整合

1. 在GitHub上创建一个repository

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427194648.png)

2. 获取到git的链接`git@github.com:bigdataxiaohan/microservicecloud-config.git`在本地创建一个仓库

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427195034.png)

3. 在本地的仓库在创建一个application.yml&nbsp;注意保存的编码格式必要是UTF-8

    ```yaml
    spring:
      profiles:
        active:
        - dev
    ---
    spring:
      profiles: dev     #开发环境
      application: 
        name: microservicecloud-config-hphblog-dev
    ---
    spring:
      profiles: test   #测试环境
      application: 
        name: microservicecloud-config-hphblog-test
    #  请保存为UTF-8格式
    ```

4. 将本地的配置文件推送到github上去

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427200208.png)

5. 查看github上的配置信息

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427200254.png)

6. 新建一个Moudle `microservicecloud-config-3344`作为配置中心模块

    pom.xml文件修改

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <artifactId>microservicecloud</artifactId>
            <groupId>com.hph.springcloud</groupId>
            <version>1.0-SNAPSHOT</version>
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>microservicecloud-config-3344</artifactId>
        <dependencies>
            <!-- springCloud Config -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-config-server</artifactId>
            </dependency>
            <!-- 图形化监控 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
            <!-- 熔断 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-hystrix</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-eureka</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-config</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jetty</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
            </dependency>
            <!-- 热部署插件 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>springloaded</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
            </dependency>
        </dependencies>
    
    </project>
    ```

    

7. application.yml

    ```yaml
    server:
      port: 3344
    
    spring:
      application:
        name:  microservicecloud-config
      cloud:
        config:
          server:
            git:
              uri: https://github.com/bigdataxiaohan/microservicecloud-config.git #GitHub上面的git仓库名字
    ```

8. Config_3344_StartSpringCloudApp

    ```java
    package com.hph.springcloud;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.config.server.EnableConfigServer;
    
    @SpringBootApplication
    @EnableConfigServer
    public class Config_3344_StartSpringCloudApp
    {
      public static void main(String[] args)
      {
       SpringApplication.run(Config_3344_StartSpringCloudApp.class,args);
      }
    }
    ```

9. windows下修改hosts文件，增加映射

    ```properties
    127.0.0.1  config-3344.com
    ```


### 测试

1. 启动Config_3344_StartSpringCloudApp

2. 访问<http://config-3344.com:3344/application-dev.yml>

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427204401.png)

3. 访问<http://config-3344.com:3344/application-test.yml>

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427204514.png)

4. 访问<http://config-3344.com:3344/application-xxx.yml>(不存在的配置)

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427204606.png)

### 访问方式

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427205614.png)

链接地址：<https://github.com/spring-cloud/spring-cloud-config/issues/292>

#### 方式一

`/{application}/{profile}[/{label}]`

访问:<http://config-3344.com:3344/application/dev/master>

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427205902.png)

访问：<http://config-3344.com:3344/application/test/master>

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427210036.png)

访问： <http://config-3344.com:3344/application/xxx/master>

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427210117.png)

#### 方式二

`/{label}/{application}-{profile}.yml`

访问<http://config-3344.com:3344/master/application-dev.yml>

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427210323.png)

访问<http://config-3344.com:3344/master/application-test.yml>

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427210349.png)

## 配置客户端

创建一个microservicecloud-config-client.yml文件

### microservicecloud-config-client.yml

```yaml
spring:
  profiles:
    active:
    - dev
---
server: 
  port: 8201 
spring:
  profiles: dev
  application: 
    name: microservicecloud-config-client
eureka: 
  client: 
    service-url: 
      defaultZone: http://eureka-dev.com:7001/eureka/   
---
server: 
  port: 8202 
spring:
  profiles: test
  application: 
    name: microservicecloud-config-client
eureka: 
  client: 
    service-url: 
      defaultZone: http://eureka-test.com:7001/eureka/
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427211654.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427223239.png)

### 新建microservicecloud-config-client-3355

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.hph.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-config-client-3355</artifactId>
    <dependencies>
        <!-- SpringCloud Config客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>

</project>
```

### bootstrap.yml

applicaiton.yml是用户级的资源配置项

```yaml
spring:
  cloud:
    config:
      name: microservicecloud-config-client #需要从github上读取的资源名称，注意没有yml后缀名
      profile: test   #本次访问的配置项
      label: master   
      uri: http://config-3344.com:3344  #本微服务启动后先去找3344号服务，通过SpringCloudConfig获取GitHub的服务地址
 
```

bootstrap.yml是系统级的，优先级更加高

### application.yml

```yaml
spring:
  application:
    name: microservicecloud-config-client
```

Spring Cloud会创建一个`Bootstrap Context`，作为Spring应用的`Application Context`的父上下文。初始化的时候，`Bootstrap Context`负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的`Environment`。`Bootstrap`属性有高优先级，默认情况下，它们不会被本地配置覆盖。 `Bootstrap context`和`Application Context`有着不同的约定，
所以新增了一个`bootstrap.yml`文件，保证`Bootstrap Context`和`Application Context`配置的分离。

增加映射

```properties
127.0.0.1  client-config.com
```

新建rest类，验证是否能从GitHub上读取配置

```java
package com.hph.springcloud.rest;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class ConfigClientRest {

    @Value("${spring.application.name}")
    private String applicationName;

    @Value("${eureka.client.service-url.defaultZone}")
    private String eurekaServers;

    @Value("${server.port}")
    private String port;

    @RequestMapping("/config")
    public String getConfig()
    {
        String str = "applicationName: "+applicationName+"\t eurekaServers:"+eurekaServers+"\t port: "+port;
        System.out.println("******str: "+ str);
        return "applicationName: "+applicationName+"\t eurekaServers:"+eurekaServers+"\t port: "+port;
    }
}
```

## 测试

启动Config配置中心3344微服务并自测

访问<http://config-3344.com:3344/application-dev.yml>

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427223505.png)

因为我们配置的文件是test访问的端口是8202 

访问测试端口

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190427223908.png)

## SpringCloud Config配置实战

注意:<font color= red>编码方式一定要是UTF-8</font>

### 新建microservicecloud-config-eureka-client.yml

```yaml
spring: 
  profiles: 
    active: 
    - dev
---
server: 
  port: 7001 #注册中心占用7001端口,冒号后面必须要有空格
   
spring: 
  profiles: dev
  application:
    name: microservicecloud-config-eureka-client
    
eureka: 
  instance: 
    hostname: eureka7001.com #冒号后面必须要有空格
  client: 
    register-with-eureka: false #当前的eureka-server自己不注册进服务列表中
    fetch-registry: false #不通过eureka获取注册信息
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/
---
server: 
  port: 7001 #注册中心占用7001端口,冒号后面必须要有空格
   
spring: 
  profiles: test
  application:
    name: microservicecloud-config-eureka-client
    
eureka: 
  instance: 
    hostname: eureka7001.com #冒号后面必须要有空格
  client: 
    register-with-eureka: false #当前的eureka-server自己不注册进服务列表中
    fetch-registry: false #不通过eureka获取注册信息
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/
```

### 新建microservicecloud-config-dept-client.yml

```yaml
spring: 
  profiles:
    active:
    - dev
--- 
server:
  port: 8001
spring: 
   profiles: dev
   application: 
    name: microservicecloud-config-dept-client
   datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: jdbc:mysql://192.168.1.110:3306/cloudDB01
    username: root
    password: 123456
    dbcp2:
      min-idle: 5
      initial-size: 5
      max-total: 5
      max-wait-millis: 200 
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml
  type-aliases-package: com.hph.springcloud.entities
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml
 
eureka: 
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    instance-id: dept-8001.com
    prefer-ip-address: true
 
info:
  app.name: hphblog-microservicecloud-springcloudconfig01
  company.name: www.hphnlog.cn
  build.artifactId: ${project.artifactId}
  build.version: ${project.version}
---
server:
  port: 8001
spring: 
   profiles: test
   application: 
    name: microservicecloud-config-dept-client
   datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: jdbc:mysql://192.168.1.110:3306/cloudDB02
    username: root
    password: 123456
    dbcp2:
      min-idle: 5
      initial-size: 5
      max-total: 5
      max-wait-millis: 200  
  
  
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml
  type-aliases-package: com.hph.springcloud.entities
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml
 
eureka: 
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    instance-id: dept-8001.com
    prefer-ip-address: true
 
info:
  app.name: hphblog-microservicecloud-springcloudconfig01
  company.name: www.hphnlog.cn
  build.artifactId: ${project.artifactId}
  build.version: ${project.version}
```

### 新建工程microservicecloud-config-eureka-client-7001

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.hph.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-config-eureka-client-7001</artifactId>
    <dependencies>
        <!-- SpringCloudConfig配置 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
        <!-- 热部署插件 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
    
</project>
```

#### bootstrap.yml

```yaml
spring:
  cloud:
    config:
      name: microservicecloud-config-eureka-client     #需要从github上读取的资源名称，注意没有yml后缀名
      profile: dev
      label: master
      uri: http://config-3344.com:3344      #SpringCloudConfig获取的服务地址
```

#### application.yml

```yaml
spring:
  application:
    name: microservicecloud-config-eureka-client
```

#### Config_Git_EurekaServerApplication

```java
package com.hph.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class Config_Git_EurekaServerApplication 
{
  public static void main(String[] args) 
  {
   SpringApplication.run(Config_Git_EurekaServerApplication.class, args);
  }
}
```

### 测试

1. 先启动microservicecloud-config-3344微服务，保证Config总配置状态可用

2. 再启动microservicecloud-config-eureka-client-7001微服务

3. 访问<http://eureka7001.com:7001/> 

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190428091607.png)

### microservicecloud-config-dept-client-8001

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.hph.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-config-dept-client-8001</artifactId>

    <dependencies>
        <!-- SpringCloudConfig配置 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>com.hph.springcloud</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>

</project>
```

#### bootstrap.yml

```yaml
spring:
  cloud:
    config:
      name: microservicecloud-config-dept-client #需要从github上读取的资源名称，注意没有yml后缀名
      #profile配置是什么就取什么配置dev or test
      #profile: dev
      profile: test
      label: master
      uri: http://config-3344.com:3344  #SpringCloudConfig获取的服务地址
```

#### application.yml

```yaml
spring:
  application:
    name: microservicecloud-config-dept-client
```

#### 主启动类

```java
package com.hph.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient //本服务启动后会自动注册进eureka服务中
@EnableDiscoveryClient //服务发现
public class DeptProvider8001_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class, args);
    }
}
```

#### 其他配置

参考microservicecloud-provider-dept-8001

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190428095701.png)

​    

### 测试

1. 启动microservicecloud-config-3344主启动类

2. 启动microservicecloud-config-client-3355主启动类

3. 启动microservicecloud-config-eureka-client-7001主启动类

4. 启动microservicecloud-config-dept-client-8001主启动类

    由于我们激活的配置是test所以我们访问一下<<http://localhost:8001/dept/list>>

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190428100653.png)

5. 修改microservicecloud-config-dept-client-8001中的profile属性改成dev

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190428102056.png)

    

6. 我们将本地的文件修改一下让dev模式下访问3号数据库重新启动所有服务

    ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/SpringConfig/20190428102753.png)

7. ![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/SpringCloud/架构/20190428121518.png)



### 完整代码

Github地址: &nbsp;<https://github.com/bigdataxiaohan/microservicecloud/tree/master/SpringConfig>










