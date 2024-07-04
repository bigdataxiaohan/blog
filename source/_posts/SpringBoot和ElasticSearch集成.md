---
title: SpringBoot和Elasticsearch集成
date: 2019-04-12 23:55:22
tags: Elasticsearch
categories: SpringBoot 
---

![AqUQhD.png](https://s2.ax1x.com/2019/04/12/AqUQhD.png)

![AqUbHx.png](https://s2.ax1x.com/2019/04/12/AqUbHx.png)

## 依赖

在Maven的pom文件中

```xml
  	 <!--SpringBoot默认使用SpringData ElasticSearch模块进行操作-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

## 自动配置

![AqDcDJ.png](https://s2.ax1x.com/2019/04/12/AqDcDJ.png)

![AqDovD.png](https://s2.ax1x.com/2019/04/12/AqDovD.png)

在SpringBoot中默认支持两种技术来和ES进行数据交互操作:

1.Jest(默认不生效的)

![AqrGPx.png](https://s2.ax1x.com/2019/04/12/AqrGPx.png)

需要导入Jest的工具包` io.searchbox.client.JestClient;`

2.SpringData ElasticSearch

```java
	@Bean
	@ConditionalOnMissingBean
	//链接es的客户端
	public Client elasticsearchClient() {
		try {
			return createClient();
		}
		catch (Exception ex) {
			throw new IllegalStateException(ex);
		}
	}
	//创建客户端
	private Client createClient() throws Exception {
        //获得每一个节点的信息
		if (StringUtils.hasLength(this.properties.getClusterNodes())) {
			return createTransportClient();
		}
		return createNodeClient();
	}
	//创创建连接点客户端
	private Client createNodeClient() throws Exception {
		Settings.Builder settings = Settings.settingsBuilder();
		for (Map.Entry<String, String> entry : DEFAULTS.entrySet()) {
			if (!this.properties.getProperties().containsKey(entry.getKey())) {
				settings.put(entry.getKey(), entry.getValue());
			}
		}
		settings.put(this.properties.getProperties());
		Node node = new NodeBuilder().settings(settings)
				.clusterName(this.properties.getClusterName()).node();
		this.releasable = node;
		return node.client();
	}

	private Client createTransportClient() throws Exception {
		TransportClientFactoryBean factory = new TransportClientFactoryBean();
		factory.setClusterNodes(this.properties.getClusterNodes());
		factory.setProperties(createProperties());
		factory.afterPropertiesSet();
		TransportClient client = factory.getObject();
		this.releasable = client;
		return client;
	}

	private Properties createProperties() {
		Properties properties = new Properties();
		properties.put("cluster.name", this.properties.getClusterName());
		properties.putAll(this.properties.getProperties());
		return properties;
	}

	@Override
	public void destroy() throws Exception {
		if (this.releasable != null) {
			try {
				if (logger.isInfoEnabled()) {
					logger.info("Closing Elasticsearch client");
				}
				try {
					this.releasable.close();
				}
				catch (NoSuchMethodError ex) {
					// Earlier versions of Elasticsearch had a different method name
					ReflectionUtils.invokeMethod(
							ReflectionUtils.findMethod(Releasable.class, "release"),
							this.releasable);
				}
			}
			catch (final Exception ex) {
				if (logger.isErrorEnabled()) {
					logger.error("Error closing Elasticsearch client: ", ex);
				}
			}
		}
	}
```

```java
@Configuration
@ConditionalOnClass({ Client.class, ElasticsearchTemplate.class })
@AutoConfigureAfter(ElasticsearchAutoConfiguration.class)
public class ElasticsearchDataAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean
	@ConditionalOnBean(Client.class)
    //操作es
	public ElasticsearchTemplate elasticsearchTemplate(Client client,
			ElasticsearchConverter converter) {
		try {
			return new ElasticsearchTemplate(client, converter);
		}
		catch (Exception ex) {
			throw new IllegalStateException(ex);
		}
	}

	@Bean
	@ConditionalOnMissingBean
	public ElasticsearchConverter elasticsearchConverter(
			SimpleElasticsearchMappingContext mappingContext) {
		return new MappingElasticsearchConverter(mappingContext);
	}

	@Bean
	@ConditionalOnMissingBean
	public SimpleElasticsearchMappingContext mappingContext() {
		return new SimpleElasticsearchMappingContext();
	}

}

```

```java
@Configuration
//启用ElasticsearchRepository接口
@ConditionalOnClass({ Client.class, ElasticsearchRepository.class })
@ConditionalOnProperty(prefix = "spring.data.elasticsearch.repositories",
		name = "enabled", havingValue = "true", matchIfMissing = true)
@ConditionalOnMissingBean(ElasticsearchRepositoryFactoryBean.class)
@Import(ElasticsearchRepositoriesRegistrar.class)
public class ElasticsearchRepositoriesAutoConfiguration {

}

```

```java
@NoRepositoryBean
public interface ElasticsearchRepository<T, ID extends Serializable> extends ElasticsearchCrudRepository<T, ID> {

	<S extends T> S index(S entity);

	Iterable<T> search(QueryBuilder query);

	Page<T> search(QueryBuilder query, Pageable pageable);

	Page<T> search(SearchQuery searchQuery);

	Page<T> searchSimilar(T entity, String[] fields, Pageable pageable);

	void refresh();

	Class<T> getEntityClass();
}
```

我们需要编写一个一个ElasticsearchRepository的子接口来操作ES

## Jest

我们需要将`spring-boot-starter-data-elasticsearch`注释掉在maven中添加jest依赖由于ES版本为5所以我们引入最新版的Jest

```java
<dependency>
   <groupId>io.searchbox</groupId>
   <artifactId>jest</artifactId>
   <version>5.3.3</version>
</dependency>
```

![Aqsh6O.png](https://s2.ax1x.com/2019/04/12/Aqsh6O.png)

引入依赖成功.

```java
protected HttpClientConfig createHttpClientConfig() {
		HttpClientConfig.Builder builder = new HttpClientConfig.Builder(
            	//getUris比较重要
				this.properties.getUris());
    	//用户名
		if (StringUtils.hasText(this.properties.getUsername())) {
			builder.defaultCredentials(this.properties.getUsername(),
					this.properties.getPassword());
		}
    	//主机
		String proxyHost = this.properties.getProxy().getHost();
		if (StringUtils.hasText(proxyHost)) {
            //主机端口
			Integer proxyPort = this.properties.getProxy().getPort();
			Assert.notNull(proxyPort, "Proxy port must not be null");
			builder.proxy(new HttpHost(proxyHost, proxyPort));
		}
		Gson gson = this.gsonProvider.getIfUnique();
		if (gson != null) {
			builder.gson(gson);
		}
		builder.multiThreaded(this.properties.isMultiThreaded());
		builder.connTimeout(this.properties.getConnectionTimeout())
				.readTimeout(this.properties.getReadTimeout());
		customize(builder);
		return builder.build();
	}

	private void customize(HttpClientConfig.Builder builder) {
		if (this.builderCustomizers != null) {
			for (HttpClientConfigBuilderCustomizer customizer : this.builderCustomizers) {
				customizer.customize(builder);
			}
		}
	}
```

```java
@ConfigurationProperties(prefix = "spring.elasticsearch.jest")
public class JestProperties {

	/**
	 * Comma-separated list of the Elasticsearch instances to use.
	 */
	private List<String> uris = new ArrayList<String>(
        	//默认与本机的9200进行交互 我们只需要在配置文件中配置spring.elasticsearch.jest.urls
			Collections.singletonList("http://localhost:9200"));

	private String username;

	private String password;

	private boolean multiThreaded = true;

	private int connectionTimeout = 3000;

	private int readTimeout = 3000;

	private final Proxy proxy = new Proxy();

	public List<String> getUris() {
		return this.uris;
	}

	public void setUris(List<String> uris) {
		this.uris = uris;
	}

	public String getUsername() {
		return this.username;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	public String getPassword() {
		return this.password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public boolean isMultiThreaded() {
		return this.multiThreaded;
	}

	public void setMultiThreaded(boolean multiThreaded) {
		this.multiThreaded = multiThreaded;
	}

	public int getConnectionTimeout() {
		return this.connectionTimeout;
	}

	public void setConnectionTimeout(int connectionTimeout) {
		this.connectionTimeout = connectionTimeout;
	}

	public int getReadTimeout() {
		return this.readTimeout;
	}

	public void setReadTimeout(int readTimeout) {
		this.readTimeout = readTimeout;
	}

	public Proxy getProxy() {
		return this.proxy;
	}

	public static class Proxy {

		private String host;

		private Integer port;

		public String getHost() {
			return this.host;
		}

		public void setHost(String host) {
			this.host = host;
		}

		public Integer getPort() {
			return this.port;
		}

		public void setPort(Integer port) {
			this.port = port;
		}

	}

}

```

在application.properties中配置

```properties
spring.elasticsearch.jest=http://192.168.1.110:9200
```

启动SpringBoot项目

![AqyVjU.png](https://s2.ax1x.com/2019/04/12/AqyVjU.png)

数据连接池已经更换。

### 保存文档

```java
package com.hph.elasticsearch;

import com.hph.elasticsearch.bean.Article;
import io.searchbox.client.JestClient;
import io.searchbox.core.Index;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.IOException;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootElasticsearchApplicationTests {

    @Autowired
    JestClient jestClient;

    @Test
    public void savDocument() {
        Article article = new Article();
        article.setId(1);
        article.setTitle("SpringBoot和Elasticsearch集成");
        article.setAuthor("清风笑丶");
        article.setContent("关于SpringBoot和Elasticsearch集成的文章");

        //构架一个索引功能
        Index index = new Index.Builder(article).index("springboot").type("javaweb").build();

        try {
            //执行
            jestClient.execute(index);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 注意

ElasticSearch引擎是大小写敏感的，强制性要求索引名和文档类型小写，对于字段名，ElasticSearch引擎会将首字母小写，建议在配置索引，文档类型和字段名时，都使用小写字母。

![AqvB0e.png](https://s2.ax1x.com/2019/04/13/AqvB0e.png)

```java
   //测试搜索
    @Test
    public void search() {
        String rule = "{\n" +
                "    \"query\" : {\n" +
                "        \"match\" : {\n" +
                "            \"content\" : \"Elasticsearch\"\n" +
                "        }\n" +
                "    }\n" +
                "}";

        //构建搜索功能
        Search search = new Search.Builder(rule).addIndex("springboot").addType("javaweb").build();

        try {
            SearchResult searchResult = jestClient.execute(search);
            System.out.println(searchResult.getJsonString());
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```

![AqvhnS.png](https://s2.ax1x.com/2019/04/13/AqvhnS.png)

这里只简要叙述一下Jest的用法更多资料请点击[Jest](https://github.com/searchbox-io/Jest/tree/master/jest))链接

## SpringData

首先我们要引入SPringleData

```xml
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

配置application.properties

```properties
spring.elasticsearch.jest.uris=http://192.168.1.110:9200

spring.data.elasticsearch.cluster-name="elasticsearch"
spring.data.elasticsearch.cluster-nodes=192.168.1.110:9300
```

![AqxM9I.png](https://s2.ax1x.com/2019/04/13/AqxM9I.png)

运行报错

![Aqx1jf.png](https://s2.ax1x.com/2019/04/13/Aqx1jf.png)

该项目中引入的ES版本为2.4.6因此我们需要更换一下ES的版本。[方法链接](https://github.com/spring-projects/spring-data-elasticsearch)

版本对比

| spring data elasticsearch | elasticsearch |
| ------------------------- | ------------- |
| 3.2.x                     | 6.5.0         |
| 3.1.x                     | 6.2.2         |
| 3.0.x                     | 5.5.0         |
| 2.1.x                     | 2.4.0         |
| 2.0.x                     | 2.2.0         |
| 1.3.x                     | 1.5.2         |

![AqxruT.png](https://s2.ax1x.com/2019/04/13/AqxruT.png)

如果版本不适配,我们可以升级SpringBoot或者安装合适版本的ES

这里我们安装一下适配版本的ES

![AqztsK.png](https://s2.ax1x.com/2019/04/13/AqztsK.png)

![AqzBid.png](https://s2.ax1x.com/2019/04/13/AqzBid.png)

版本更换过来了

![AqzrRI.png](https://s2.ax1x.com/2019/04/13/AqzrRI.png)

### 数据准备

```java
package com.hph.elasticsearch.bean;

import org.springframework.data.elasticsearch.annotations.Document;

@Document(indexName = "hphblog",type = "javaweb")
public class Blog {
    private  Integer id;
    private  String  blogName;
    private  String author;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getBlogName() {
        return blogName;
    }

    public void setBlogName(String blogName) {
        this.blogName = blogName;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    @Override
    public String toString() {
        return "Blog{" +
                "id=" + id +
                ", blogName='" + blogName + '\'' +
                ", author='" + author + '\'' +
                '}';
    }
}

```

`SpringBootElasticsearchApplicationTests`中添加saveIndex方法然后运行

```java
@Autowired
BlogRepository blogRepository;

@Test
public void saveIndex() {
    Blog blog = new Blog();
    blogRepository.index(blog);
}
```

![ALSkwD.png](https://s2.ax1x.com/2019/04/13/ALSkwD.png)

### 存储数据

```java
    @Test
    public void saveData(){
        Blog blog = new Blog();
        blogRepository.index(blog);
        blog.setId(1);
        blog.setBlogName("SpringBoot和Elasticsearch的学习");
        blog.setAuthor("清风笑丶");
        blogRepository.index(blog);

    }
```

运行该方法

数据已经存入。

![ALSNpn.png](https://s2.ax1x.com/2019/04/13/ALSNpn.png)

### 自定义查询

在BlogRepository自定义方法。

```java
public interface BlogRepository extends ElasticsearchRepository<Blog,Integer> {
    public List<Book> findByBlogNameLike (String blogName);
}
```

在测试类中

```java
    @Test
    public void searchBlogBynName(){
        for (Blog blog : blogRepository.findByBlogNameLike("学")) {
            System.out.println(blog);

        }
    }
```

![ALSvB8.png](https://s2.ax1x.com/2019/04/13/ALSvB8.png)

想了解更多使用方法可以点击[链接](https://docs.spring.io/spring-data/elasticsearch/docs/3.1.6.RELEASE/reference/html/#elasticsearch.query-methods)来阅读更多使用方法。









