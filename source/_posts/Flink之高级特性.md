---
title: Flink高级特性
date: 2020-09-12 18:08:40
tags: Flink
categories: 大数据
---

本文主要介绍了Flink的一些特性，比如异步IO的使用，分布式缓存和流批处理中广播变量的应用。


在使用Flink处理数据的过程中，往往需要和外部的系统进行交互，通常情况下可以使用MapFunction创建外部链接，将请求发送到外部存储，IO阻塞，等待请求返回，然后继续发送下一个请求。这种方式在网络上耗费了大量的时间而且会影响到外部系统的性能，可以通过增加MapFunction来提升效率，加大资源开销。这种场景主要出现在维度表的Join上。

### Async I/O

Flink 在1.2中引入了Async I/O，在异步模式下，将IO操作异步化，单个并行可以连续发送多个请求，哪个请求先返回就先处理，从而在连续的请求间不需要阻塞式等待，大大提高了流处理效率。但前提是数据库本身需要支持异步客户端。

Async I/O 是阿里巴巴贡献给社区的一个呼声非常高的特性，解决与外部系统交互时网络延迟的系统瓶颈的问题。通过使用Async I/O 可以很大程度上提升Flink系统的吞吐量，这是因为异步函数可以尽可能的一部并发查询外部的数据库，或者异步调用接口，在异步IO中需要考虑查询超时和并发线程控制两个因素，Async I/O提供了timeout和capacity，timeout为Async I/O最大等待时间，如果超过该事件Flink认为查询请求失败，对于超时的请求可以复写timeout处理，capacity是同一时间点的异步请求并发数，在capacity被请求消耗完后，Flink直接出发反压机制来抑制上游数据接入，从而使Flink任务正常运行。

使用异步IO方式进行数据输出，其输出结果的先后顺序有可能并不是按照之前原有数据的顺序进行排序，在Flink中，需要用户显式指定是否对结果排序输出，而是否排序同样影响着结果的顺序和系统性能，下面针对结果是否进行排序输出进行对比：

- 乱序模式：异步查询结果输出中，原本数据元素的顺序可能会发生变化，请求一旦完成就会输出结果，可以使用AsyncDataStream.unorderedWait(...)方法应用这种模式。如果系统同时选择使用Process Time特征具有最低的延时和负载。

- 顺序模式：异步查询结果将按照输入数据元素的顺序输出，原本Stream数据元素的顺序保持不变，这种情况下具有较高的时延和负载，因为结果数据需要在Operator的Buffer中进行缓存，直到所有异步请求处理完毕，将按照原来元素顺序输出结果，这也将对Checkpointing过程造成额外的延时和性能损耗。可以使用AsyncDataStream.orderedWait(...)方法使用这种模式。

在使用Event-Time时间概念处理流数据的过程中，Asynchronous I/OOperator总能够正确保持Watermark的顺序，即使使用乱序模式，输出Watermark也会保持原有顺序，但对于在Watermark之间的数据元素则不保持原来的顺序，也就是说如果使用了Watermark，将会对异步IO造成一定的时延和开销，具体取决于Watermark的频率，频率越高时延越高同时开销越大。

这段代码为写入数据的Bean对象

```java
package com.hph.bean;

public class UserLocation {
    private String user;
    private long timestamp;
    private String lng;
    private String lat;
    private String address;


    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public UserLocation(String user, long timestamp, String lng, String lat, String address) {
        this.user = user;
        this.timestamp = timestamp;
        this.lng = lng;
        this.lat = lat;
        this.address = address;
    }


    public UserLocation() {
    }

    public String getUser() {
        return user;
    }

    public void setUser(String user) {
        this.user = user;
    }

    public long getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(long timestamp) {
        this.timestamp = timestamp;
    }

    public String getLng() {
        return lng;
    }

    public void setLng(String lng) {
        this.lng = lng;
    }

    public String getLat() {
        return lat;
    }

    public void setLat(String lat) {
        this.lat = lat;
    }

    @Override
    public String toString() {
        return "UserLocation{" +
                "user='" + user + '\'' +
                ", timestamp=" + timestamp +
                ", lng='" + lng + '\'' +
                ", lat='" + lat + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

```

这段代码为JSON字符串转为Java对象

```java
package com.hph.bean;

import java.util.List;

public class Location {


    /**
     * status : 1
     * regeocode : {"addressComponent":{"city":"沧州市","province":"河北省","adcode":"130929","district":"献县","towncode":"130929213000","streetNumber":{"number":[],"direction":[],"distance":[],"street":[]},"country":"中国","township":"垒头乡","businessAreas":[[]],"building":{"name":[],"type":[]},"neighborhood":{"name":[],"type":[]},"citycode":"0317"},"formatted_address":"河北省沧州市献县垒头乡天运平衡块厂"}
     * info : OK
     * infocode : 10000
     */

    private String status;
    /**
     * addressComponent : {"city":"沧州市","province":"河北省","adcode":"130929","district":"献县","towncode":"130929213000","streetNumber":{"number":[],"direction":[],"distance":[],"street":[]},"country":"中国","township":"垒头乡","businessAreas":[[]],"building":{"name":[],"type":[]},"neighborhood":{"name":[],"type":[]},"citycode":"0317"}
     * formatted_address : 河北省沧州市献县垒头乡天运平衡块厂
     */

    private RegeocodeBean regeocode;
    private String info;
    private String infocode;

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public RegeocodeBean getRegeocode() {
        return regeocode;
    }

    public void setRegeocode(RegeocodeBean regeocode) {
        this.regeocode = regeocode;
    }

    public String getInfo() {
        return info;
    }

    public void setInfo(String info) {
        this.info = info;
    }

    public String getInfocode() {
        return infocode;
    }

    public void setInfocode(String infocode) {
        this.infocode = infocode;
    }

    public static class RegeocodeBean {
        /**
         * city : 沧州市
         * province : 河北省
         * adcode : 130929
         * district : 献县
         * towncode : 130929213000
         * streetNumber : {"number":[],"direction":[],"distance":[],"street":[]}
         * country : 中国
         * township : 垒头乡
         * businessAreas : [[]]
         * building : {"name":[],"type":[]}
         * neighborhood : {"name":[],"type":[]}
         * citycode : 0317
         */

        private AddressComponentBean addressComponent;
        private String formatted_address;

        public AddressComponentBean getAddressComponent() {
            return addressComponent;
        }

        public void setAddressComponent(AddressComponentBean addressComponent) {
            this.addressComponent = addressComponent;
        }

        public String getFormatted_address() {
            return formatted_address;
        }

        public void setFormatted_address(String formatted_address) {
            this.formatted_address = formatted_address;
        }

        public static class AddressComponentBean {
            private String city;
            private String province;
            private String adcode;
            private String district;
            private String towncode;
            private StreetNumberBean streetNumber;
            private String country;
            private String township;
            private BuildingBean building;
            private NeighborhoodBean neighborhood;
            private String citycode;
            private List<List<?>> businessAreas;

            public String getCity() {
                return city;
            }

            public void setCity(String city) {
                this.city = city;
            }

            public String getProvince() {
                return province;
            }

            public void setProvince(String province) {
                this.province = province;
            }

            public String getAdcode() {
                return adcode;
            }

            public void setAdcode(String adcode) {
                this.adcode = adcode;
            }

            public String getDistrict() {
                return district;
            }

            public void setDistrict(String district) {
                this.district = district;
            }

            public String getTowncode() {
                return towncode;
            }

            public void setTowncode(String towncode) {
                this.towncode = towncode;
            }

            public StreetNumberBean getStreetNumber() {
                return streetNumber;
            }

            public void setStreetNumber(StreetNumberBean streetNumber) {
                this.streetNumber = streetNumber;
            }

            public String getCountry() {
                return country;
            }

            public void setCountry(String country) {
                this.country = country;
            }

            public String getTownship() {
                return township;
            }

            public void setTownship(String township) {
                this.township = township;
            }

            public BuildingBean getBuilding() {
                return building;
            }

            public void setBuilding(BuildingBean building) {
                this.building = building;
            }

            public NeighborhoodBean getNeighborhood() {
                return neighborhood;
            }

            public void setNeighborhood(NeighborhoodBean neighborhood) {
                this.neighborhood = neighborhood;
            }

            public String getCitycode() {
                return citycode;
            }

            public void setCitycode(String citycode) {
                this.citycode = citycode;
            }

            public List<List<?>> getBusinessAreas() {
                return businessAreas;
            }

            public void setBusinessAreas(List<List<?>> businessAreas) {
                this.businessAreas = businessAreas;
            }

            public static class StreetNumberBean {
                private List<?> number;
                private List<?> direction;
                private List<?> distance;
                private List<?> street;

                public List<?> getNumber() {
                    return number;
                }

                public void setNumber(List<?> number) {
                    this.number = number;
                }

                public List<?> getDirection() {
                    return direction;
                }

                public void setDirection(List<?> direction) {
                    this.direction = direction;
                }

                public List<?> getDistance() {
                    return distance;
                }

                public void setDistance(List<?> distance) {
                    this.distance = distance;
                }

                public List<?> getStreet() {
                    return street;
                }

                public void setStreet(List<?> street) {
                    this.street = street;
                }
            }

            public static class BuildingBean {
                private List<?> name;
                private List<?> type;

                public List<?> getName() {
                    return name;
                }

                public void setName(List<?> name) {
                    this.name = name;
                }

                public List<?> getType() {
                    return type;
                }

                public void setType(List<?> type) {
                    this.type = type;
                }
            }

            public static class NeighborhoodBean {
                private List<?> name;
                private List<?> type;

                public List<?> getName() {
                    return name;
                }

                public void setName(List<?> name) {
                    this.name = name;
                }

                public List<?> getType() {
                    return type;
                }

                public void setType(List<?> type) {
                    this.type = type;
                }
            }
        }
    }
}
```

Kafka模拟数据

````java
package com.hph.datasource.producer;


import java.math.BigDecimal;

import com.apifan.common.random.source.DateTimeSource;
import com.apifan.common.random.source.OtherSource;
import com.google.gson.Gson;
import com.hph.bean.UserLocation;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.time.LocalDateTime;
import java.util.Properties;

public class AsyncDataProducer {


    //以济南市范围为例
    //北 117.033334,36.685959
    // 南 117.029382,36.667087
    // 西 117.017668,36.673745
    // 东 117.043826,36.67554

    private static double chinaWest = 115.7958606;
    private static double chinaEast = 117.83072785;
    private static double chinaSouth = 37.89179029;
    private static double chinaNorth = 42.36503919;
    private static Gson gson;

    //117.033945,36.684657

    public static void main(String[] args) throws InterruptedException {
        Properties props = new Properties();
        // Kafka服务端的主机名和端口号
        props.put("bootstrap.servers", "hadoop102:9092,hadoop103:9092,hadoop104:9092");
        // 等待所有副本节点的应答
        props.put("acks", "all");
        // 消息发送最大尝试次数
        props.put("retries", 0);
        // 一批消息处理大小
        props.put("batch.size", 16384);
        // 请求延时
        props.put("linger.ms", 1);
        // 发送缓存区内存大小
        props.put("buffer.memory", 33554432);
        // key序列化
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        // value序列化
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        Producer<String, String> producer = new KafkaProducer<>(props);


        String user = OtherSource.getInstance().randomPlateNumber(true);
        LocalDateTime begin = LocalDateTime.of(2020, 3, 12, 0, 0, 0);
        LocalDateTime end = LocalDateTime.of(2020, 9, 12, 23, 30, 0);
        gson = new Gson();
        for (int i = 0; i <= 300; i++) {
            long timestamp = DateTimeSource.getInstance().randomTimestamp(begin, end) / 1000;
            UserLocation userLocation = new UserLocation(user, timestamp, randomLonLat(true), randomLonLat(false), "");
            System.out.println(userLocation);
            producer.send(new ProducerRecord<String, String>("car_location", gson.toJson(userLocation)));
            Thread.sleep(1000);
        }
    }

    private static String randomLonLat(boolean isLongitude) {
        if (isLongitude) {
            BigDecimal bigDecimal = new BigDecimal(Math.random() * (chinaEast - chinaWest) + chinaWest);
            return bigDecimal.setScale(5, BigDecimal.ROUND_HALF_UP).toString();
        } else {
            BigDecimal bigDecimal = new BigDecimal(Math.random() * (chinaNorth - chinaSouth) + chinaSouth);
            return bigDecimal.setScale(6, BigDecimal.ROUND_HALF_UP).toString();
        }
    }


}

````

Flink异步IO的相关功能实现。

```java
package com.hph.async;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.google.gson.Gson;
import com.hph.bean.UserLocation;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.async.ResultFuture;
import org.apache.flink.streaming.api.functions.async.RichAsyncFunction;
import org.apache.http.HttpResponse;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.nio.client.CloseableHttpAsyncClient;
import org.apache.http.impl.nio.client.HttpAsyncClients;
import org.apache.http.util.EntityUtils;

import java.util.Collections;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.Future;
import java.util.function.Supplier;

public class AsyncGetLocation extends RichAsyncFunction<String, UserLocation> {
    private static Gson gson;
    private static String address;
    private transient CloseableHttpAsyncClient httpClient;


    @Override
    public void open(Configuration parameters) throws Exception {
        super.open(parameters);
        //初始化异步的HTTPClient
        RequestConfig requestConfig = RequestConfig.custom()
                .setSocketTimeout(3000)
                .setConnectTimeout(3000)
                .build();
        httpClient = HttpAsyncClients.custom()
                .setMaxConnPerRoute(20)
                .setDefaultRequestConfig(requestConfig).build();
        httpClient.start();
    }

    @Override
    public void asyncInvoke(String input, ResultFuture<UserLocation> resultFuture) throws Exception {
        gson = new Gson();
        UserLocation userLocation = gson.fromJson(input, UserLocation.class);
        String lng = userLocation.getLng();
        String lat = userLocation.getLat();

        String url = "https://restapi.amap.com/v3/geocode/regeo?key=8726dd1c181748cdd953c6bebbe14de9&location=" + lng + "," + lat;
        HttpGet httpGet = new HttpGet(url);

        Future<HttpResponse> future = httpClient.execute(httpGet, null);

        CompletableFuture.supplyAsync(new Supplier<String>() {
            @Override
            public String get() {
                try {
                    HttpResponse response = future.get();
                    if (response.getStatusLine().getStatusCode() == 200) {
                        String result = EntityUtils.toString(response.getEntity());
                        //这里引入了FastJson，主要是因为调用高德地图接口时出现Gson解析异常
                        JSONObject jsonObject = JSON.parseObject(result);
                        //获取位置信息
                        JSONObject regeocode = jsonObject.getJSONObject("regeocode");
                        if (regeocode != null && !regeocode.isEmpty()) {
                            //获取地址位置
                            address = regeocode.getString("formatted_address");
                        }
                    }
                    return address;
                } catch (Exception e) {
                    return "结果获取失败";
                }
            }
        }).thenAccept((String address) -> {
            resultFuture.complete(Collections.singleton(new UserLocation(userLocation.getUser(), userLocation.getTimestamp(), userLocation.getLng(), userLocation.getLat(), address)));
        });
    }
}

```

Flink异步IO的程序入口

```java
package com.hph.async;

import com.hph.bean.UserLocation;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.AsyncDataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;

import java.util.Properties;
import java.util.concurrent.TimeUnit;

public class AsyncApp {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        Properties props = new Properties();
        //指定kafka的Broker地址
        props.setProperty("bootstrap.servers", "hadoop102:9092,hadoop103:9092,hadoop104:9092");
        //设置组ID
        props.setProperty("group.id", "flink");
        props.setProperty("auto.offset.reset", "earliest");
        //kafka自动提交偏移量，
        props.setProperty("enable.auto.commit", "false");
        // key序列化
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        // value序列化
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        DataStreamSource<String> carLocation = environment.addSource(new FlinkKafkaConsumer<>(
                "car_location",
                new SimpleStringSchema(),
                props
        ));

        SingleOutputStreamOperator<UserLocation> beas = AsyncDataStream.orderedWait(carLocation, new AsyncGetLocation(), 0, TimeUnit.MILLISECONDS, 100);
        beas.print();


        environment.execute("Async Demo");

    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200913094228.gif)

### 分布式缓存

在批计算中，处理的数据集大部分来自文件，如果数据存放在HDFS等分布式文件系统上，Flink无法像MapReduce一样让计算在数据所在的位置上进行，就会出现网路频繁复制文件，Spark中cache主要是为了复用RDD，在Flink中主要是将高频使用的文件通过分布式缓存的方式，将其放置在taskmanager中，避免了读取某些文件必须通过网络从而开销提升效率。

在分布式缓存在StreamExecutionEnvironment直接注册文件或者文件件，Flink启动任务的过程中会将这些文件同步到taskmanager中，通过设置Boolean参数制定文件是否可执行。

在这段代码中我是用到了泰坦尼克号的数据集作为分布式缓存的文件

```java
package com.hph.cache;

import org.apache.commons.io.FileUtils;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.io.File;
import java.util.HashMap;
import java.util.List;


public class DistributedCache {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment executionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<Long> data = executionEnvironment.generateSequence(1, 891);
        
        executionEnvironment.registerCachedFile("D:\\数据集\\Titanic-master\\train.csv", "train", true);

        SingleOutputStreamOperator<Tuple2<Long, String>> result = data.map(new RichMapFunction<Long, Tuple2<Long, String>>() {
            private HashMap<Long, String> cacheDataMap = new HashMap<>();

            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);

                File train = getRuntimeContext().getDistributedCache().getFile("train");
                List<String> lists = FileUtils.readLines(train);
                for (String list : lists) {
        /*             PassengerId,Survived,Pclass,Name,Sex,Age,SibSp,Parch,Ticket,Fare,Cabin,Embarked
                       1,0,3,"Braund, Mr. Owen Harris",male,22,1,0,A/5 21171,7.25,,S     */

                    String[] data = list.split(",");
                    //缓存id 对应名字
                    cacheDataMap.put(Long.valueOf(data[0]), data[3] + "," + data[4]);
                }
            }

            @Override
            public Tuple2<Long, String> map(Long value) throws Exception {
                return Tuple2.of(value, cacheDataMap.get(value));
            }
        });
        
        result.print();
        executionEnvironment.execute();

    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200914232906.png)

在Flink任务的启动过程中我们可以发现，Flink中的taskmanager已经获取到了改文件，并将文件缓存到内存中，这样子就可以在这个进程中访问内存获取到改数据。因此减轻了网络开销，提升了性能。

### 广播变量

广播变量是一种常用的数据共享方式，是对像数据集通过网络传输，在每个计算节点上都存储一份该数据集，计算节点可以直接从本地内存中读取数据集，避免了多次通过网络读取，从而提升行能，在Spark中基本上会用在大小表Join，将较小RDD中的数据直接通过collect算子拉取到Driver端的内存中来，然后对其创建一个Broadcast变量；这种在Spark中只适合于广播变量较小的文件(几百MB或者1~2GB)左右。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200913121949.png)

在Flink中对广播变量的使用和分布式缓存基本类似，只不过在Flink中广播变量没有Spark使用起来那么方便。首先我们如果想要在流逝文件中动态的获取数据库中的数据作为广播变量的话，这里我选择的是Redis，目前基本上是自定义Source读取Redis中的数据，将数据写出并添加broadcast作为标记，同时需要创建MapStateDescriptor来描述我们的广播变量。感觉比较心塞目前好像只能是用Flink批处理才能实现广播变量。不过我们也可以通过一些措施实现流式的广播变量，在官方文档中给定的是关于State的广播变量案例，关于Flink的State后续会讲到。这里我们依批处理的为例。

```java
package com.hph.broadcast;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import redis.clients.jedis.Jedis;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class BroadcastBatch {

    public static void main(String[] args) throws Exception {

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        Jedis jedis = new Jedis("hadoop102", 6379);
        Map<String, String> redisResultMap = jedis.hgetAll("broadcast");
        ArrayList<Map<String, String>> broadCastBD = new ArrayList<>();

        broadCastBD.add(redisResultMap);

        DataSet<Map<String, String>> bdDataSet = env.fromCollection(broadCastBD);

        //源数据  需要与广播变量关联。
        MapOperator<Long, String> data = env.generateSequence(1, 891).map(new MapFunction<Long, String>() {
            @Override
            public String map(Long value) throws Exception {
                return String.valueOf(value);
            }
        });

        对数据进行操作
        MapOperator<String, Tuple2<String, String>> result = data.map(new RichMapFunction<String, Tuple2<String, String>>() {
            List<HashMap<String, String>> broadCastMap = new ArrayList<HashMap<String, String>>();
            HashMap<String, String> allMap = new HashMap<String, String>();

            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);
                this.broadCastMap = getRuntimeContext().getBroadcastVariable("broadCastMapName");
                for (HashMap map : broadCastMap) {
                    allMap.putAll(map);
                }
            }

            @Override
            public Tuple2<String, String> map(String value) throws Exception {
                return Tuple2.of(String.valueOf(value), allMap.get(value));
            }
        }).withBroadcastSet(bdDataSet, "broadCastMapName");
        result.print();


    }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200914233739.png)

数据关联正确，在Flink中广播变量的使用相较于Spark还是有些复杂的，那么对于流式的数据我们该怎么处理呢

```java
package com.hph.broadcast;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.state.MapStateDescriptor;
import org.apache.flink.api.common.typeinfo.BasicTypeInfo;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.BroadcastStream;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.BroadcastProcessFunction;
import org.apache.flink.streaming.api.functions.source.RichSourceFunction;
import org.apache.flink.util.Collector;
import redis.clients.jedis.Jedis;


import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.TimeUnit;

/**
 * 广播流读取Redis
 */

public class BroadcastStreamStream {

    public static void main(String[] args) throws Exception {

        // 构建流处理环境
        final StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();

        // 配置处理环境的并发度为4
        environment.setParallelism(4);
        final MapStateDescriptor<String, String> REDIS_BROADCAST = new MapStateDescriptor<>(
                "redis-broadCast",
                BasicTypeInfo.STRING_TYPE_INFO,
                BasicTypeInfo.STRING_TYPE_INFO);

        // 自定义广播流（单例）
        BroadcastStream<Map<String, String>> broadcastStream = environment.addSource(new RichSourceFunction<Map<String, String>>() {

            private volatile boolean isRunning = true;
            private volatile Jedis jedis;
            private volatile Map<String, String> map = new ConcurrentHashMap(16);

            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);
                jedis = new Jedis("hadoop102", 6379);

            }

            /**
             * 数据源：模拟每30秒重新读取redis中的相关数据
             * @param ctx
             * @throws Exception
             */

            @Override
            public void run(SourceContext<Map<String, String>> ctx) throws Exception {
                while (isRunning) {
                    long newTime = System.currentTimeMillis();
                    //每间隔30秒查询一次Redis的数据作为广播变量
                    TimeUnit.SECONDS.sleep(30);
                    map = jedis.hgetAll("broadcast");
                    ctx.collect(map);
                    map.clear();
                }
            }

            @Override
            public void cancel() {
                isRunning = false;
            }
        }).setParallelism(1).broadcast(REDIS_BROADCAST);


        //生成测试数据
        DataStreamSource<Long> data = environment.generateSequence(1, 891);
        DataStream<String> dataStream = data.map(new MapFunction<Long, String>() {
            @Override
            public String map(Long value) throws Exception {
                //数据每间隔5s发送一次
                Thread.sleep(5000);
                return String.valueOf(value);
            }
        });

        // 数据流和广播流连接处理并将拦截结果打印
        dataStream.connect(broadcastStream).process(new BroadcastProcessFunction<String, Map<String, String>, String>() {
            private volatile int i;
            private volatile Map<String, String> keywords;
            private volatile Jedis jedis;

            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);

                if (keywords == null) {
                    jedis = new Jedis("hadoop102", 6379);
                    Map<String, String> resultMap = jedis.hgetAll("broadcast");
                    keywords = resultMap;
                }
            }

            @Override
            public void processBroadcastElement(Map<String, String> value, Context ctx, Collector<String> out) throws Exception {
                //todo 获取的Map的数据
                keywords = value;

            }

            @Override
            public void processElement(String value, ReadOnlyContext ctx, Collector<String> out) throws Exception {
                out.collect("数据源中的ID:" + value + ", 广播变量中获取到Value：" + keywords.get(value));
            }

        }).print();

        environment.execute();
    }

}
```



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200914235115.gif)

数据关联正确，在Flink中使用广播变量还是稍微复杂了一下，需要使用RichMapFunction接口，在open()方法中调用 getRuntimeContext().getBroadcastVariable("") 获取到数据集;同时也需要我们对数据转换为集合。

