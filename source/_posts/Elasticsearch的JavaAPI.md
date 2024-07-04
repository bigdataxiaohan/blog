---
title: Elasticsearch的JavaAPI
date: 2018-12-16 18:44:50
tags: Elasticsearch
categories: 数据库
---

## 获取客户端对象

```java
public class App {

    private TransportClient client;

    //获取客户端对象
    @Before
    public void getClinet() throws UnknownHostException {
        Settings settings = Settings.builder().put("cluster.name", "my-application").build();

        //获得客户端对象
        client = new PreBuiltTransportClient(settings);

        client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("192.168.1.11"), 9300));
        System.out.println(client.toString());

    }
```

	## 创建索引

```java
//创建索引
@Test
public void createIndex() {
    client.admin().indices().prepareCreate("blog1").get();
    client.close();
}

//删除索引
@Test
public void deleteIndex() {
    client.admin().indices().prepareDelete("blog").get();
    client.close();
}
```

## 新建文档

```java
//新建文档
@Test
public void createIndexByJson() {
    //创建文档内容

    String json = "{" + "\"id\":\"1\"," + "\"title\":\"基于Lucene的搜索服务器\","
            + "\"content\":\"它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口\"" + "}";

    //创建
    IndexResponse response = client.prepareIndex("blog", "article", "1").setSource(json).execute().actionGet();
    //打印返回值
    System.out.println("索引" + response.getIndex());
    System.out.println("类型" + response.getType());
    System.out.println("id" + response.getId());
    System.out.println("结果" + response.getResult());
    client.close();
}

//创建文档以hashmap
@Test
public void createIndexBymap() {
    HashMap<String, Object> json = new HashMap<String, Object>();

    json.put("id", "2");
    json.put("title", "hph");
    json.put("content", "博客 hph.blog");
    IndexResponse response = client.prepareIndex("blog", "article", "2").setSource(json).execute().actionGet();
    //打印返回值
    System.out.println("索引" + response.getIndex());
    System.out.println("类型" + response.getType());
    System.out.println("id" + response.getId());
    System.out.println("结果" + response.getResult());
    client.close();
}
//创建文档以hashmap
    @Test
    public void createIndexBymap() {
        HashMap<String, Object> json = new HashMap<String, Object>();

        json.put("id", "2");
        json.put("title", "hph");
        json.put("content", "博客 hph.blog");
        IndexResponse response = client.prepareIndex("blog", "article", "2").setSource(json).execute().actionGet();
        //打印返回值
        System.out.println("索引" + response.getIndex());
        System.out.println("类型" + response.getType());
        System.out.println("id" + response.getId());
        System.out.println("结果" + response.getResult());
        client.close();

    }

    //创建文档已bulder
    @Test
    public void createIndexbyBulider() throws IOException {
        XContentBuilder builder = XContentFactory.jsonBuilder().startObject().field("id", "4").field("title", "博客").field("content", "hphblog.cn").endObject();
        IndexResponse response = client.prepareIndex("blog", "article", "3").setSource(builder).execute().actionGet();
        //打印返回值
        System.out.println("索引" + response.getIndex());
        System.out.println("类型" + response.getType());
        System.out.println("id" + response.getId());
        System.out.println("结果" + response.getResult());
        client.close();
    }
```

## 查询索引

```java
//单个索引查询
@Test
public void queryIndex() {
    //查询
    GetResponse response = client.prepareGet("blog", "article", "2").get();

    //打印
    System.out.println(response.getSourceAsString());

    //关闭资源
    client.close();

}

//多个索引查询
@Test
public void queryMultiIndex() {

    //查询
    MultiGetResponse response = client.prepareMultiGet().add("blog", "article", "3")
            .add("blog", "article", "1")
            .add("blog", "article", "2").get();

    for (MultiGetItemResponse multiGetItemResponse : response) {
        GetResponse response1 = multiGetItemResponse.getResponse();

        //判断是否存在
        if (response1.isExists()) {
            System.out.println(response1.getSourceAsString());
        }
    }
}

//更改数据
@Test
public void update() throws ExecutionException, InterruptedException, IOException {
    UpdateRequest updateRequest = new UpdateRequest("blog", "article", "2");
    updateRequest.doc(XContentFactory.jsonBuilder().startObject().field("id", "2")
            .field("title", "hphblog").field("content", "大数据博客学习技术分享").endObject()
    );

    UpdateResponse response = client.update(updateRequest).get();

    //打印返回值
    System.out.println("索引" + response.getIndex());
    System.out.println("类型" + response.getType());
    System.out.println("id" + response.getId());
    System.out.println("结果" + response.getResult());
    client.close();

}
```

## 更新文档

```java
//更新文档updaset
@Test
public void upsert() throws IOException, ExecutionException, InterruptedException {
    //没有就创建
    IndexRequest indexRequest = new IndexRequest("blog", "article", "5");
    indexRequest.source(XContentFactory.jsonBuilder().startObject().field("id", "5")
            .field("title", "大数据技术分享的学习").field("content", "大数据技术的分享与学习希望和大家多多交流").endObject
                    ());
    //有文档内容就更新
    UpdateRequest updateRequest = new UpdateRequest("blog", "article", "5");
    updateRequest.doc(XContentFactory.jsonBuilder().startObject().field("id", "5")
            .field("title", "hphblog").field("content", "技术分享、技术交流、技术学习").endObject()
    );

    updateRequest.upsert(indexRequest);

    UpdateResponse response = client.update(updateRequest).get();

    //打印返回值
    System.out.println("索引" + response.getIndex());
    System.out.println("类型" + response.getType());
    System.out.println("id" + response.getId());
    System.out.println("结果" + response.getResult());
    client.close();


}
```

## 删除文档

```java
//删除文档
@Test
public void delete() {
    client.prepareDelete("blog", "article", "5").get();
    client.close();

}
```

## 查询所有

```java
//查询所有
@Test
public void matchAllquery() {
    //1.执行查询
    SearchResponse response = client.prepareSearch("blog").setTypes("article").setQuery(QueryBuilders.matchAllQuery()).get();

    SearchHits hits = response.getHits();

    System.out.println("查询的结果为" + hits.getTotalHits());

    Iterator<SearchHit> iterator = hits.iterator();
    while (iterator.hasNext()) {
        SearchHit next = iterator.next();
        System.out.println(next.getSourceAsString());
    }

    client.close();
}

//分词查询
@Test
public void query() {
    //1.执行查询
    SearchResponse response = client.prepareSearch("blog").setTypes("article").setQuery(QueryBuilders.queryStringQuery("学习全文")).get();

    SearchHits hits = response.getHits();

    System.out.println("查询的结果为" + hits.getTotalHits());

    Iterator<SearchHit> iterator = hits.iterator();
    while (iterator.hasNext()) {
        SearchHit next = iterator.next();
        System.out.println(next.getSourceAsString());
    }

    client.close();
}
```

## 通配符查询

```java
//通配符查询
@Test
public void wildcardQuery() {
    //1.执行查询
    SearchResponse response = client.prepareSearch("blog").setTypes("article").setQuery(QueryBuilders.wildcardQuery("content", "分")).get();

    SearchHits hits = response.getHits();

    System.out.println("查询的结果为" + hits.getTotalHits());

    Iterator<SearchHit> iterator = hits.iterator();
    while (iterator.hasNext()) {
        SearchHit next = iterator.next();
        System.out.println(next.getSourceAsString());
    }

    client.close();
}
```

## 模糊查询

```java
//模糊查询
@Test
public void fuzzyQuery() {
    //1.执行查询
    SearchResponse response = client.prepareSearch("blog").setTypes("article").setQuery(QueryBuilders.fuzzyQuery("title", "LuceNe")).get();

    SearchHits hits = response.getHits();

    System.out.println("查询的结果为" + hits.getTotalHits());

    Iterator<SearchHit> iterator = hits.iterator();
    while (iterator.hasNext()) {
        SearchHit next = iterator.next();
        System.out.println(next.getSourceAsString());
    }

    client.close();
}

//映射相关的操作
@Test
public void createMapping() throws Exception {

    // 1设置mapping
    XContentBuilder builder = XContentFactory.jsonBuilder()
            .startObject()
            .startObject("article")
            .startObject("properties")
            .startObject("id1")
            .field("type", "string")
            .field("store", "yes")
            .endObject()
            .startObject("title2")
            .field("type", "string")
            .field("store", "no")
            .endObject()
            .startObject("content")
            .field("type", "string")
            .field("store", "yes")
            .endObject()
            .endObject()
            .endObject()
            .endObject();
```

## 添加mapping

```java
    // 2 添加mapping
    PutMappingRequest mapping = Requests.putMappingRequest("blog1").type("article").source(builder);

    client.admin().indices().putMapping(mapping).get();

    // 3 关闭资源
    client.close();
}
```