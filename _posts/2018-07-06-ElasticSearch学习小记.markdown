---
layout: post
title:  "ElasticSearch学习小记"
date:   2018-06-26 11:39:34
categories: 技术
tags: git
description: 本文主要介绍使用java-API客户端工具jest调用elasticsearch的基本方法，附有多篇学习文档和博客。
---

### 参考文档
* [Elasticsearch: 权威指南,官方文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)
* [ElasticSearch: Index 和 Type 的区别以及6.0新变化](http://www.unclewang.info/learn/es/332/)

### Java客户端Jest操作自学文档
* jest客户端提供API服务学习参考文档
	* [Jest官方文档地址](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)
	* [JAVA客户端之Jest操作详解](https://blog.csdn.net/u011781521/article/details/77852861?locationNum=7&fps=1)
	* [elasticsearch-jest-example](https://github.com/ameizi/elasticsearch-jest-example/blob/master/src/main/java/net/aimeizi/client/elasticsearch/TransportClient.java)
	* [Elasticsearch Jest实战深入详解](https://blog.csdn.net/laoyang360/article/details/77146063)
	* [Elasticsearch之Java客户端Jest](https://blog.csdn.net/vinegar93/article/details/53223819/)
	* [ElasticSearch AggregationBuilders java api常用聚会查询](https://www.cnblogs.com/xionggeclub/p/7975982.html)

> 本文讲述的内容是基于已经建立好elasticsearch数据源的基础之上做的一些数据操作。包括增、删、改、查等基础操作。此外本文所有关于elasticsearch的操作是基于java客户端 jest操作api实现。

### 一、MVN依赖引入
* 第一步先将以下依赖文件加入本地pom中，引入依赖jar。

```
<!--zsearch start-->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpcore</artifactId>
    <version>4.4</version>
</dependency>
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpmime</artifactId>
    <version>4.3.2</version>
</dependency>
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.4.1</version>
</dependency>

<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.9</version>
</dependency>

<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpasyncclient</artifactId>
    <version>4.1.3</version>
</dependency>

<dependency>
    <groupId>io.searchbox</groupId>
    <artifactId>jest</artifactId>
    <version>5.3.2</version>
</dependency>

<dependency>
    <groupId>io.searchbox</groupId>
    <artifactId>jest-common</artifactId>
    <version>5.3.2</version>
</dependency>

<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpcore-nio</artifactId>
    <version>4.3.2</version>
</dependency>

<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>5.3.0</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-queries</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-core</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-sandbox</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-analyzers-common</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-backward-codecs</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-memory</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-highlighter</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-queryparser</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-join</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-spatial</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-expressions</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.9.9</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.11.0</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.11.0</version>
</dependency>

<!--zsearch end-->

```

### Jest操作API指南

* Jest客户端初始化

```java

private String        SERVER_URI = "xxxx";
private String        HTTP_HOST  = "xxxx";
private String        USERNAME   = "xxxx";
private String        password   = "xxxx";
private String        indexName  = "xxxx";
private String        typeName   = "xxxx";


JestClientFactory factory = new JestClientFactory();
HttpClientConfig httpClientConfig = new HttpClientConfig.Builder(SERVER_URI)
    .setPreemptiveAuth(new HttpHost(HTTP_HOST)).defaultCredentials(USERNAME, password)
    .multiThreaded(true).defaultMaxTotalConnectionPerRoute(50).maxTotalConnection(50)
    .build();
factory.setHttpClientConfig(httpClientConfig);
JestClient client = factory.getObject();
```

* 基本操作封装

```java
/**
 * 创建索引
 * @param jestClient
 * @param indexName
 * @return
 * @throws Exception
 */
public boolean createIndex(JestClient jestClient, String indexName) throws Exception {

    JestResult jr = jestClient.execute(new CreateIndex.Builder(indexName).build());
    return jr.isSucceeded();
}

/**
 * Put映射
 * @param jestClient
 * @param indexName
 * @param typeName
 * @param source
 * @return
 * @throws Exception
 */
public boolean createIndexMapping(JestClient jestClient, String indexName, String typeName,
                                  String source) throws Exception {

    PutMapping putMapping = new PutMapping.Builder(indexName, typeName, source).build();
    JestResult jr = jestClient.execute(putMapping);
    return jr.isSucceeded();
}

/**
 * Get映射
 * @param jestClient
 * @param indexName
 * @param typeName
 * @return
 * @throws Exception
 */
public String getIndexMapping(JestClient jestClient, String indexName, String typeName)
                                                                                       throws Exception {

    GetMapping getMapping = new GetMapping.Builder().addIndex(indexName).addType(typeName)
        .build();
    JestResult jr = jestClient.execute(getMapping);
    return jr.getJsonString();
}

/**
 * 索引文档
 * @param jestClient
 * @param indexName
 * @param typeName
 * @param objs
 * @return
 * @throws Exception
 */
public boolean index(JestClient jestClient, String indexName, String typeName, List<Object> objs)
                                                                                                 throws Exception {

    Bulk.Builder bulk = new Bulk.Builder().defaultIndex(indexName).defaultType(typeName);
    for (Object obj : objs) {
        Index index = new Index.Builder(obj).build();
        bulk.addAction(index);
    }
    BulkResult br = jestClient.execute(bulk.build());
    return br.isSucceeded();

}

/**
 * 搜索文档
 * @param jestClient
 * @param indexName
 * @param typeName
 * @param query
 * @return
 * @throws Exception
 */
public SearchResult search(JestClient jestClient, String indexName, String typeName,
                           String query) throws Exception {

    Search search = new Search.Builder(query).addIndex(indexName).addType(typeName).build();
    return jestClient.execute(search);
}

/**
 * Count文档
 * @param jestClient
 * @param indexName
 * @param typeName
 * @param query
 * @return
 * @throws Exception
 */
public Double count(JestClient jestClient, String indexName, String typeName, String query)
                                                                                           throws Exception {

    Count count = new Count.Builder().addIndex(indexName).addType(typeName).query(query)
        .build();
    CountResult results = jestClient.execute(count);
    return results.getCount();
}

/**
 * Get文档
 * @param jestClient
 * @param indexName
 * @param typeName
 * @param id
 * @return
 * @throws Exception
 */
public JestResult get(JestClient jestClient, String indexName, String typeName, String id)
                                                                                          throws Exception {

    Get get = new Get.Builder(indexName, id).type(typeName).build();
    return jestClient.execute(get);
}

/**
 * Delete索引
 * @param jestClient
 * @param indexName
 * @return
 * @throws Exception
 */
public boolean delete(JestClient jestClient, String indexName) throws Exception {

    JestResult jr = jestClient.execute(new DeleteIndex.Builder(indexName).build());
    return jr.isSucceeded();
}

/**
 * Delete文档
 * @param jestClient
 * @param indexName
 * @param typeName
 * @param id
 * @return
 * @throws Exception
 */
public boolean delete(JestClient jestClient, String indexName, String typeName, String id)
                                                                                          throws Exception {

    DocumentResult dr = jestClient.execute(new Delete.Builder(id).index(indexName)
        .type(typeName).build());
    return dr.isSucceeded();
}

/**
 * 关闭JestClient客户端
 * @param jestClient
 * @throws Exception
 */
public void closeJestClient(JestClient jestClient) throws Exception {
    if (jestClient != null) {
        jestClient.shutdownClient();
    }
}

```

* 测试调用

```java
/**
 * 建表
 * @throws Exception
 */
@Test
public void createIndex() throws Exception {

    boolean result = createIndex(client, indexName + "_test");
    System.out.println(result);
}

/**
 * 建映射关系
 * @throws Exception
 */
@Test
public void createIndexMapping() throws Exception {

    String source = "{\""
                    + typeName
                    + "\":{\"properties\":{"
                    + "\"id\":{\"type\":\"integer\"}"
                    + ",\"name\":{\"type\":\"string\",\"index\":\"not_analyzed\"}"
                    + ",\"birth\":{\"type\":\"date\",\"format\":\"strict_date_optional_time||epoch_millis\"}"
                    + "}}}";
    System.out.println(source);
    boolean result = createIndexMapping(client, indexName + "_test", typeName, source);
    System.out.println(result);
}

/**
 * 查看索引结构
 * @throws Exception
 */
@Test
public void getIndexMapping() throws Exception {
    String mapping = getIndexMapping(client, indexName + "_test", typeName);
    System.out.println(mapping);
}

/**
 * 删表
 * @throws Exception
 */
@Test
public void deleteIndex() throws Exception {

    boolean result = delete(client, indexName + "_test");
    System.out.println(result);
}

/**
 * 精确查找
 */
@Test
public void queryStringTest() throws Exception{

    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    QueryBuilder queryBuilder = QueryBuilders
            .termQuery("mobile", "18667151234");//单值完全匹配查询
    searchSourceBuilder.query(queryBuilder);
    // 分页
    searchSourceBuilder.size(10);
    searchSourceBuilder.from(0);
    String query = searchSourceBuilder.toString();
    System.out.println(query);
    SearchResult result = search(client, indexName, typeName, query);
    List<JsonObject> objs = result.getSourceAsObjectList(JsonObject.class, true);
    for (JsonObject obj : objs) {
        System.out.println(obj.toString());
    }
}


/**
 * 精确查找+排序
 */
@Test
public void queryStringOrderTest() throws Exception{

    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    QueryBuilder queryBuilder = QueryBuilders
            .termQuery("mobile", "18667151234");//单值完全匹配查询
    searchSourceBuilder.query(queryBuilder);
    // 分页
    searchSourceBuilder.size(10);
    searchSourceBuilder.from(0);

    // 排序
    SortBuilder sortBuilder = SortBuilders.fieldSort("gmtOccur").order(SortOrder.ASC);
    searchSourceBuilder.sort(sortBuilder);

    String query = searchSourceBuilder.toString();
    System.out.println(query);
    SearchResult result = search(client, indexName, typeName, query);
    List<JsonObject> objs = result.getSourceAsObjectList(JsonObject.class, true);
    for (JsonObject obj : objs) {
        System.out.println(obj.toString());
    }
}


/**
 * 聚合搜索
 */
@Test
public void queryStringArggrationTest() throws Exception{

    TermsAggregationBuilder teamAgg= AggregationBuilders.terms("mobile_count").field("mobile");

    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

    //QueryBuilder queryBuilder = QueryBuilders
    //        .termQuery("mobile", "18667151234");//单值完全匹配查询
    //searchSourceBuilder.query(queryBuilder);

    searchSourceBuilder.aggregation(teamAgg);
    // 分页
    searchSourceBuilder.size(50);
    //searchSourceBuilder.from(0);
    String query = searchSourceBuilder.toString();
    System.out.println(query);
    SearchResult result = search(client, indexName, typeName, query);

    List<JsonObject> objs = result.getSourceAsObjectList(JsonObject.class, true);
    for (JsonObject obj : objs) {
        System.out.println(obj.toString());
    }
}



/**
 * 通配符查询
 * @throws Exception
 */
@Test
public void wildcardQueryStringTest() throws Exception {

    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    QueryBuilder queryBuilder = QueryBuilders.wildcardQuery("mobile", "186*34");
    searchSourceBuilder.query(queryBuilder);
    searchSourceBuilder.size(10);
    searchSourceBuilder.from(0);
    String query = searchSourceBuilder.toString();
    System.out.println(query);
    SearchResult result = search(client, indexName, typeName, query);
    List<JsonObject> objs = result.getSourceAsObjectList(JsonObject.class, true);
    for (JsonObject obj : objs) {
        System.out.println(obj.toString());
    }
}

/**
 * 前缀搜索
 * @throws Exception
 */
@Test
public void prefixQuery() throws Exception {

    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    QueryBuilder queryBuilder = QueryBuilders
            .prefixQuery("mobile", "1866");//前缀查询
    searchSourceBuilder.query(queryBuilder);
    searchSourceBuilder.size(10);
    searchSourceBuilder.from(0);
    String query = searchSourceBuilder.toString();
    System.out.println(query);
    SearchResult result = search(client, indexName, typeName, query);
    List<JsonObject> objs = result.getSourceAsObjectList(JsonObject.class, true);
    for (JsonObject obj : objs) {
        System.out.println(obj.toString());
    }
}

/**
 * 区间搜索
 * @throws Exception
 */
@Test
public void rangeQuery() throws Exception {

    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    QueryBuilder queryBuilder = QueryBuilders
            .rangeQuery("gmtOccur")
            .gte("2018-07-05T16:00:00")
            .lte("2018-07-05T18:40:00")
            .includeLower(true)
            .includeUpper(true);//区间查询
    searchSourceBuilder.query(queryBuilder);
    searchSourceBuilder.size(10);
    searchSourceBuilder.from(0);
    String query = searchSourceBuilder.toString();
    System.out.println(query);
    SearchResult result = search(client, indexName, typeName, query);
    List<JsonObject> objs = result.getSourceAsObjectList(JsonObject.class, true);
    for (JsonObject obj : objs) {
        System.out.println(obj.toString());
    }

}

    /**
     * 文档统计计数
     * @throws Exception
     */
@Test
public void testCountDoc() throws Exception {
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query(QueryBuilders.queryStringQuery("mobile:18667151234 "));
    double count = count(client, indexName, typeName, searchSourceBuilder.toString());
    System.out.println(count);

    searchSourceBuilder = new SearchSourceBuilder();
    QueryBuilder queryBuilder = QueryBuilders.wildcardQuery("mobile", "1866*");
    searchSourceBuilder.query(queryBuilder);
    System.out.println(searchSourceBuilder.toString());
    count = count(client, indexName, typeName, searchSourceBuilder.toString());
    System.out.println(count);
}

/**
 * 删除文档
 * @throws Exception
 */
@Test
public void testDeleteDoc() throws Exception {
    String id = "2";
    boolean result = delete(client, indexName, "default", id);
    System.out.println(result);
}

```

