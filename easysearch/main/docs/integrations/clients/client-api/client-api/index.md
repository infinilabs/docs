---
title: "API 使用"
date: 0001-01-01
summary: "Easysearch Java API Client 使用文档 #  管理索引 #  使用客户端对索引进行管理
String index = &#34;test1&#34;; if (client.indices().exists(r -&gt; r.index(index)).value()) { LOGGER.info(&#34;Deleting index &#34; + index); DeleteIndexResponse deleteIndexResponse = client.indices().delete(new DeleteIndexRequest.Builder().index(index).build()); LOGGER.info(deleteIndexResponse.toString()); } LOGGER.info(&#34;Creating index &#34; + index); CreateIndexResponse createIndexResponse = client.indices().create(req -&gt; req.index(index)); CloseIndexResponse closeIndexResponse = client.indices().close(req -&gt; req.index(index)); OpenResponse openResponse = client.indices().open(req -&gt; req.index(index)); RefreshResponse refreshResponse = client.indices().refresh(req -&gt; req.index(index)); FlushResponse flushResponse = client.indices().flush(req -&gt; req.index(index)); ForcemergeResponse forcemergeResponse = client."
---

# Easysearch Java API Client 使用文档


### 管理索引
使用客户端对索引进行管理
```java
String index = "test1";
if (client.indices().exists(r -> r.index(index)).value()) {
    LOGGER.info("Deleting index " + index);
    DeleteIndexResponse deleteIndexResponse = client.indices().delete(new DeleteIndexRequest.Builder().index(index).build());
    LOGGER.info(deleteIndexResponse.toString());

}
LOGGER.info("Creating index " + index);
CreateIndexResponse createIndexResponse = client.indices().create(req -> req.index(index));
CloseIndexResponse closeIndexResponse = client.indices().close(req -> req.index(index));
OpenResponse openResponse = client.indices().open(req -> req.index(index));
RefreshResponse refreshResponse = client.indices().refresh(req -> req.index(index));
FlushResponse flushResponse = client.indices().flush(req -> req.index(index));
ForcemergeResponse forcemergeResponse = client.indices().forcemerge(req -> req.index(index).maxNumSegments(1L));
```
也可以用异步方式执行
```java
        EasysearchAsyncClient asyncClient = SampleClient.createAsyncClient();
        asyncClient.indices().exists(req -> req.index(index)).thenCompose(exists -> {
                if (exists.value()) {
                    LOGGER.info("Deleting index " + index);
                    return asyncClient.indices().delete(r -> r.index(index)).thenAccept(deleteResponse -> {
                        LOGGER.info(deleteResponse);
                    });
                }
                return CompletableFuture.completedFuture(null);
            }).thenCompose(v -> {
                LOGGER.info("Creating index " + index);
                return asyncClient.indices().create(req -> req.index(index));

            }).whenComplete((createResponse, throwable) -> {
                if (throwable != null) {
                    LOGGER.error("Error during index operations", throwable);
                } else {
                    LOGGER.info("Index created successfully");
                }
            })
            .get(30, TimeUnit.SECONDS);

```

### 滚动索引 Rollover
滚动索引是一种管理时序数据的有效方式，当索引满足特定条件时（如大小、文档数量、年龄），会自动创建新的索引。

示例
```java
// 创建初始索引并设置别名
String index = "test-00001";
client.indices().create(req -> req.index(index).aliases("test_log", a -> a.isWriteIndex(true)));

// 配置并执行滚动
 RolloverResponse res = client.indices().rollover(req -> req
            .alias("test_log")    // 指定别名
            .conditions(c -> c
                .maxDocs(100L)    // 文档数量达到100时滚动
                .maxAge(b -> b.time("7d"))    // 索引时间达到7天时滚动
                .maxSize("5gb"))); // 索引大小达到5GB时滚动
```
### Mapping 设置
基本设置
```java
String index = "test1";
PutMappingResponse response = client.indices().putMapping(req -> req.index(index)
    .properties("field1", p -> p.keyword(k -> k))         // keyword类型
    .properties("field2", p -> p.text(t -> t))            // text类型
);
```
更完整的示例，包含多种常用字段类型
```java
response = client.indices().putMapping(req -> req.index(index)
    // 常用字段类型示例
    .properties("keyword_field", p -> p.keyword(k -> k))           // keyword 类型
    .properties("text_field", p -> p.text(t -> t))                // text 类型
    .properties("long_field", p -> p.long_(l -> l))               // long 类型
    .properties("integer_field", p -> p.integer(i -> i))          // integer 类型
    .properties("short_field", p -> p.short_(s -> s))             // short 类型
    .properties("byte_field", p -> p.byte_(b -> b))               // byte 类型
    .properties("double_field", p -> p.double_(d -> d))           // double 类型
    .properties("float_field", p -> p.float_(f -> f))             // float 类型
    .properties("date_field", p -> p.date(d -> d))                // date 类型
    .properties("boolean_field", p -> p.boolean_(b -> b))         // boolean 类型
    .properties("binary_field", p -> p.binary(b -> b))            // binary 类型
    .properties("ip_field", p -> p.ip(i -> i))                    // ip 类型
    .properties("geo_point_field", p -> p.geoPoint(g -> g))       // geo_point 类型
    .properties("flat_field", p -> p.flattened(f -> f))           // flattened 类型

    
    // date 类型
    .properties("date_field", p -> p.date(d -> d.format("yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis")))
    
    // object 类型
    .properties("object_field", p -> p.object(o -> o
            .properties("sub_field1", sp -> sp.keyword(k -> k))
            .properties("sub_field2", sp -> sp.long_(l -> l))
        )
    )
    
    // nested 类型
    .properties("nested_field", p -> p.nested(n -> n
            .properties("sub_field1", sp -> sp.keyword(k -> k))
            .properties("sub_field2", sp -> sp.text(t -> t))
        )
    )
);
```
常用字段类型配置
```java
response = client.indices().putMapping(req -> req.index(index)
        // text 类型配置
        .properties("text_field2", p -> p
            .text(t -> t
                .analyzer("standard")                    // 分析器
                .searchAnalyzer("standard")              // 搜索分析器
                .fields("raw", f -> f.keyword(k -> k))    // 添加keyword子字段
                .copyTo("other_field")                   // 复制到其他字段
            )
        )

        // keyword 类型配置
        .properties("keyword_field2", p -> p
            .keyword(k -> k
                .ignoreAbove(256)                             // 忽略超过长度的值
                .nullValue("NULL")                            // null值替换
                .docValues(true)                              // 是否开启doc_values
            )
        )

        // date 类型配置
        .properties("date_field2", p -> p
            .date(d -> d
                .format("yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis")  // 日期格式
            )
        )
);
```
### 更新索引 Settings
可以用JSON 的方式更新 特定索引的设置
```java
 String settings = "{" +
            "    \"index\": {" +
            "        \"number_of_replicas\": 2," +
            "        \"refresh_interval\": \"5s\"," +
            "        \"max_result_window\": 50000," +
            "        \"analysis\": {\" +
            "            \"analyzer\": {" +
            "                \"my_analyzer\": {" +
            "                    \"type\": \"custom\"," +
            "                    \"tokenizer\": \"standard\"," +
            "                    \"filter\": [\"lowercase\", \"asciifolding\"]" +
            "                }" +
            "            }" +
            "        }" +
            "    }" +
            "}";
            
// 如果要更新静态设置（比如分词器），需要先关闭索引
String index = "test1";
client.indices().close(c -> c.index(index));
// 更新设置
client.indices().putSettings(req -> req
    .index(index)
    .withJson(new StringReader(settings))
);
// 重新打开索引
client.indices().open(c -> c.index(index));
```
### 创建带有自定义映射和设置的索引
使用 Easysearch Java 客户端创建带有自定义映射和设置的索引。

使用 JSON 配置
```java
String settingsJson = """
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 1,
        "analysis": {
          "analyzer": {
            "my_analyzer": {
              "type": "custom",
              "tokenizer": "standard",
              "filter": ["lowercase", "asciifolding"]
            }
          }
        }
      },
      "mappings": {
        "properties": {
          "id": {
            "type": "keyword"
          },
          "title": {
            "type": "text",
            "analyzer": "my_analyzer"
          },
          "content": {
            "type": "text"
          },
          "status": {
            "type": "keyword"
          },
          "tags": {
            "type": "keyword"
          },
          "create_time": {
            "type": "date",
            "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
          },
          "update_time": {
            "type": "date",
            "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
          },
          "view_count": {
            "type": "long"
          },
          "price": {
            "type": "double"
          }
        }
      }
    }
    """;

// 创建索引
client.indices().create(req -> req
    .index("my_index")
    .withJson(new StringReader(settingsJson))
);

```

### 创建索引模板
使用 Easysearch Java 客户端创建索引模板。
```java
String templateJson = "{"
    + "  \"index_patterns\": [\"log-*\"], "
    + "  \"template\": {"
    + "    \"settings\": {"
    + "      \"number_of_shards\": 1,"
    + "      \"number_of_replicas\": 1,"
    + "      \"refresh_interval\": \"5s\","
    + "      \"analysis\": {"
    + "        \"analyzer\": {"
    + "          \"my_analyzer\": {"
    + "            \"type\": \"custom\","
    + "            \"tokenizer\": \"standard\","
    + "            \"filter\": [\"lowercase\"]"
    + "          }"
    + "        }"
    + "      }"
    + "    },"
    + "    \"mappings\": {"
    + "      \"_source\": {"
    + "        \"enabled\": true"
    + "      },"
    + "      \"properties\": {"
    + "        \"@timestamp\": {"
    + "          \"type\": \"date\""
    + "        },"
    + "        \"message\": {"
    + "          \"type\": \"text\","
    + "          \"analyzer\": \"my_analyzer\""
    + "        },"
    + "        \"level\": {"
    + "          \"type\": \"keyword\""
    + "        },"
    + "        \"service\": {"
    + "          \"type\": \"keyword\""
    + "        },"
    + "        \"trace_id\": {"
    + "          \"type\": \"keyword\""
    + "        },"
    + "        \"metrics\": {"
    + "          \"type\": \"object\","
    + "          \"properties\": {"
    + "            \"value\": {"
    + "              \"type\": \"double\""
    + "            },"
    + "            \"name\": {"
    + "              \"type\": \"keyword\""
    + "            }"
    + "          }"
    + "        }"
    + "      }"
    + "    }"
    + "  },"
    + "  \"priority\": 100"
    + "}";

// 创建或更新模板
PutIndexTemplateRequest request = PutIndexTemplateRequest.of(builder -> builder
    .name("logs-template")                 // 模板名称
    .withJson(new StringReader(templateJson))
);

client.indices().putIndexTemplate(request);
```


### Bulk 批量写入
```java
EasysearchClient client = SampleClient.create();
String json2 = "{"
        + "    \"@timestamp\": \"2023-01-08T22:50:13.059Z\","
        + "    \"agent\": {"
        + "      \"version\": \"7.3.2\","
        + "      \"type\": \"filebeat\","
        + "      \"ephemeral_id\": \"3ff1f2c8-1f7f-48c2-b560-4272591b8578\","
        + "      \"hostname\": \"ba-0226-msa-fbl-747db69c8d-ngff6\""
        + "    }"
        + "}";
        
BulkRequest.Builder br = new BulkRequest.Builder();
for (int i = 0; i < 10; i++) {
    br.operations(op -> op.index(idx -> idx.index(indexName).document(JsonData.fromJson(json2))));
}
BulkResponse bulkResponse = client.bulk(br.build());
if (bulkResponse.errors()) {
    for (BulkResponseItem item : bulkResponse.items()) {
        System.out.println(item.toString());
    }
}
```

### 索引单个文档
```java
String json2 = "{"
    + "    \"@timestamp\": \"2023-01-08T22:50:13.059Z\","
    + "    \"agent\": {"
    + "      \"version\": \"7.3.2\","
    + "      \"type\": \"filebeat\","
    + "      \"ephemeral_id\": \"3ff1f2c8-1f7f-48c2-b560-4272591b8578\","
    + "      \"hostname\": \"ba-0226-msa-fbl-747db69c8d-ngff6\""
    + "    }"
    + "}";

IndexRequest<JsonData> request = IndexRequest.of(i -> i
    .index("logs")
    .withJson(new StringReader(json2))
);
IndexResponse response = client.index(request);
System.out.println(response);

// 也可以这样
LogEntry logEntry = mapper.readValue(json2, LogEntry.class);
IndexRequest<LogEntry> request2 = IndexRequest.of(i -> i
    .index(indexName)
    .id(logEntry.getAgent().getEphemeralId())
    .document(logEntry)
);
IndexResponse response2 = client.index(request2);

// 或者这样
IndexRequest.Builder<LogEntry> indexReqBuilder = new IndexRequest.Builder<>();
indexReqBuilder.index(indexName);
indexReqBuilder.id(logEntry.getAgent().getEphemeralId());
indexReqBuilder.document(logEntry);
response2 = client.index(indexReqBuilder.build());
```

### 删除文档
```java
DeleteRequest deleteRequest = new DeleteRequest.Builder()
    .index(indexName)
    .id("3ff1f2c8-1f7f-48c2-b560-4272591b8578")
    .build();
DeleteResponse response = client.delete(deleteRequest);
```

### deleteByQuery 删除
```java
DeleteByQueryRequest deleteByQueryRequest = new DeleteByQueryRequest.Builder()
    .index(indexName)
    .query(q -> q.match(new MatchQuery.Builder()
        .field("agent.type").query("filebeat").build())
    ).build();
DeleteByQueryResponse response = client.deleteByQuery(deleteByQueryRequest);
```

### 更新文档
```java
UpdateRequest updateRequest = UpdateRequest.of(u -> u
    .index(indexName)
    .id("3ff1f2c8-1f7f-48c2-b560-4272591b8578")
    .doc(Map.of("agent.type", "logstash"))
);
UpdateResponse<Map<String, Object>> response = client.update(updateRequest, Map.class);

```

### updateByQuery 更新
```java
Query query = Query.of(q -> q
    .term(t -> t
        .field("agent.type")
        .value(v -> v.stringValue("filebeat"))
    )
);
UpdateByQueryRequest updateByQueryRequest = UpdateByQueryRequest.of(u -> u
    .index(indexName).query(query).script(s -> s.inline(in ->
        in.source("ctx._source.agent.type = params.param1")
            .lang("painless")
            .params(Map.of("param1", JsonData.of("logstash"))))).refresh(true)
);
UpdateByQueryResponse response = client.updateByQuery(updateByQueryRequest);
System.out.println(response.updated());

```

### 搜索文档
```java
Query query = Query.of(q -> q
    .term(t -> t
        .field("agent.type")
        .value(v -> v.stringValue("filebeat"))
    )
);

SortOptions.Builder sb = new SortOptions.Builder();
SortOptions sortOptions = sb.field(fs -> fs.field("@timestamp").order(SortOrder.Desc)).build();

final SearchRequest.Builder searchReq = new SearchRequest.Builder().allowPartialSearchResults(false)
    .index(indexName)
    .size(10)
    .sort(sortOptions)
    .source(sc -> sc.fetch(true))
    .trackTotalHits(tr -> tr.enabled(true))
    .query(query);

SearchResponse<LogEntry> searchResponse = client.search(searchReq.build(), LogEntry.class);
System.out.println(searchResponse.hits().total());
for (Hit<LogEntry> hit : searchResponse.hits().hits()) {
    System.out.println(JsonData.of(hit.source()).toJson(new JacksonJsonpMapper()));
}
```

### 带子聚合的日期直方图聚合

```java
SearchRequest searchRequest = SearchRequest.of(s -> s
    .index(index)
    .size(0)  // 不需要返回文档，只要聚合结果
    .aggregations("by_date", a -> a
        .dateHistogram(dh -> dh
            .field("create_time")
            .calendarInterval(CalendarInterval.Month) // 按月聚合数据
            .format("yyyy-MM-dd")
            .minDocCount(1)
        ).aggregations("avg_price", avg -> avg // 计算每月的平均价格
            .avg(a1 -> a1
                .field("price")
            )
        ).aggregations("avg_view_count", avg -> avg // 计算每月的平均浏览次数
            .avg(a1 -> a1
                .field("view_count")
            )
        ).aggregations("by_status", terms -> terms // 统计每月不同状态的文档数量
            .terms(t -> t
                .field("status")
                .size(10)
            )
        ).aggregations("price_stats", stats -> stats // 计算价格的统计信息（最小值、最大值、平均值、总和）
            .stats(s1 -> s1
                .field("price")
            )
        )
    ));
    
SearchResponse<Void> response = client.search(searchRequest, Void.class);
// 处理聚合结果
response.aggregations()
    .get("by_date")
    .dateHistogram()
    .buckets()
    .array()
    .forEach(bucket -> {
        // 基本信息
        System.out.printf("\n日期: %s (文档数: %d)\n",
            bucket.keyAsString(),
            bucket.docCount());

        // 平均值
        System.out.printf("平均价格: %.2f\n",
            bucket.aggregations().get("avg_price").avg().value());
        System.out.printf("平均浏览: %.2f\n",
            bucket.aggregations().get("avg_view_count").avg().value());

        // 价格统计
        StatsAggregate stats = bucket.aggregations().get("price_stats").stats();
        System.out.printf("价格统计: 最小值=%.2f, 最大值=%.2f, 平均值=%.2f\n",
            stats.min(), stats.max(), stats.avg());

        // 状态分布
        System.out.println("状态分布:");
        bucket.aggregations()
            .get("by_status")
            .sterms()
            .buckets()
            .array()
            .forEach(status -> System.out.printf("  %s: %d\n",
                status.key().stringValue(),
                status.docCount()));

        System.out.println("----------------------------------------");
    });
```

### Reindex 

```java
ReindexResponse response = client.reindex(r -> r
    .source(s -> s.index("test1"))
    .dest(d -> d.index("test1_new_index"))
    .script(sc -> sc
        .inline(i -> i
            .source(
                "if (ctx._source.price != null) { " +
                    "    ctx._source.price *= 1.1; " +  // 价格上调10%
                    "    ctx._source.updated = true; " +
                    "}"
            )
            .lang("painless")
        )
    )
);
```

### 异步方式 Reindex
```java

        ReindexResponse response = client.reindex(r -> r
            .source(s -> s.index("test1"))
            .dest(d -> d.index("test1_new_index"))
            .script(sc -> sc
                .inline(i -> i
                    .source(
                        "if (ctx._source.price != null) { " +
                            "    ctx._source.price *= 1.1; " +  // 价格上调10%
                            "    ctx._source.updated = true; " +
                            "}"
                    )
                    .lang("painless")
                )
            ).waitForCompletion(false)
        );
        String taskId = response.task();
        System.out.println("Started reindex task: " + taskId);
        // 监控任务进度
        boolean completed = false;
        while (!completed) {
            try {
                Thread.sleep(1000); // 每1秒检查一次
                GetTasksResponse taskResponse = client.tasks().get(g -> g.taskId(taskId).waitForCompletion(false));
                Info taskInfo = taskResponse.task();
                if (taskInfo == null) {
                    // 任务可能已完成
                    System.out.println("Task completed or not found");
                    break;
                }

                // 获取任务状态
                JsonData status = taskInfo.status();
                if (status != null) {
                    System.out.println("Running time in millis: " + taskInfo.runningTimeInNanos() / 1_000_000L);
                    System.out.println("Current status: " + status.toJson());
                }

                if (taskResponse.completed()) {
                    completed = true;
                    System.out.println("Reindex completed successfully");

                    // 获取结果
                    JsonData result = taskInfo.status();
                    if (result != null) {
                        // 解析状态信息中的统计数据
                        String resultStr = result.toJson().toString();
                        System.out.println("Final result: " + resultStr);
                    }
                }

            } catch (Exception e) {
                e.printStackTrace();
                break;
            }
        }
```



### 附测试用例
```java
public class APITest {

    EasysearchClient client;
    ObjectMapper mapper = new ObjectMapper();
    String indexName = "test1";

    {
        try {
            client = SampleClient.create();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    
    
    // 创建索引并通过 JSON 设置 mappings 和 settings
    @Test
    public void testCreateIndexByJSON() throws IOException {
        String index = "test1";
        ESVersionInfo version = client.info().version();
        LOGGER.info("Server: " + version.number());
        if (client.indices().exists(r -> r.index(index)).value()) {
            LOGGER.info("Deleting index " + index);
            DeleteIndexResponse deleteIndexResponse = client.indices().delete(new DeleteIndexRequest.Builder().index(index).build());
            LOGGER.info(deleteIndexResponse.toString());

            LOGGER.info("Creating index " + index);

            // Settings and mappings in JSON format
            String settingsJson = "{\n" +
                "                  \"settings\": {\n" +
                "                    \"number_of_shards\": 1,\n" +
                "                    \"number_of_replicas\": 1,\n" +
                "                    \"analysis\": {\n" +
                "                      \"analyzer\": {\n" +
                "                        \"my_analyzer\": {\n" +
                "                          \"type\": \"custom\",\n" +
                "                          \"tokenizer\": \"standard\",\n" +
                "                          \"filter\": [\"lowercase\", \"asciifolding\"]\n" +
                "                        }\n" +
                "                      }\n" +
                "                    }\n" +
                "                  },\n" +
                "                  \"mappings\": {\n" +
                "                    \"properties\": {\n" +
                "                      \"id\": {\n" +
                "                        \"type\": \"keyword\"\n" +
                "                      },\n" +
                "                      \"title\": {\n" +
                "                        \"type\": \"text\",\n" +
                "                        \"analyzer\": \"my_analyzer\"\n" +
                "                      },\n" +
                "                      \"content\": {\n" +
                "                        \"type\": \"text\"\n" +
                "                      },\n" +
                "                      \"status\": {\n" +
                "                        \"type\": \"keyword\"\n" +
                "                      },\n" +
                "                      \"tags\": {\n" +
                "                        \"type\": \"keyword\"\n" +
                "                      },\n" +
                "                      \"create_time\": {\n" +
                "                        \"type\": \"date\",\n" +
                "                        \"format\": \"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis\"\n" +
                "                      },\n" +
                "                      \"update_time\": {\n" +
                "                        \"type\": \"date\",\n" +
                "                        \"format\": \"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis\"\n" +
                "                      },\n" +
                "                      \"view_count\": {\n" +
                "                        \"type\": \"long\"\n" +
                "                      },\n" +
                "                      \"price\": {\n" +
                "                        \"type\": \"double\"\n" +
                "                      }\n" +
                "                    }\n" +
                "                  }\n" +
                "                }";

            // Create index using JSON string
            CreateIndexResponse createIndexResponse = client.indices().create(req -> req
                .index(index)
                .withJson(new StringReader(settingsJson))
            );

            LOGGER.info(createIndexResponse.toString());
        }
    }

    @Test
    public void testCatHealth() throws IOException {
        ESVersionInfo version = client.info().version();
        System.out.println("Server: " + version.number());
        HealthResponse res = client.cat().health();
        System.out.println(res.toString());
    }

    @Test
    public void testCatIndices() throws IOException {
        ESVersionInfo version = client.info().version();
        System.out.println("Server: " + version.number());
        IndicesResponse res = client.cat().indices();
        System.out.println(res.toString());
    }

    @Test
    public void testCatNodes() throws IOException {
        ESVersionInfo version = client.info().version();
        System.out.println("Server: " + version.number());
        NodesResponse res = client.cat().nodes();
        System.out.println(res.toString());
    }


    @Test
    public void testScrollQuery() throws IOException {
        SortOptions.Builder sb = new SortOptions.Builder();
        SortOptions sortOptions = sb.field(fs -> fs.field("@timestamp").order(SortOrder.Desc)).build();

        Time time = Time.of(t -> t.time("1m"));
        final SearchRequest.Builder searchReq = new SearchRequest.Builder().allowPartialSearchResults(false)
            .index(Collections.singletonList(indexName)).size(10).sort(sortOptions).source(sc -> sc.fetch(true))
            .trackTotalHits(tr -> tr.enabled(true)).scroll(time).query(q -> q.matchAll(new MatchAllQuery.Builder().build()));

        SearchResponse<LogEntry> searchResponse = client.search(searchReq.build(), LogEntry.class);
        String scrollId = null;
        scrollId = searchResponse.scrollId();
        System.out.println(searchResponse.hits().total());
        for (Hit<LogEntry> hit : searchResponse.hits().hits()) {
            System.out.println("_id " + hit.id());
        }

        while (true) {
            String finalScrollId = scrollId;
            ScrollResponse<LogEntry> scrollResponse = client.scroll(sc -> sc.scrollId(finalScrollId).scroll(time), LogEntry.class);
            if (scrollResponse.hits().hits().isEmpty()) {
                break;
            }
            for (Hit<LogEntry> hit : scrollResponse.hits().hits()) {
                System.out.println("_id " + hit.id());
            }
            scrollId = scrollResponse.scrollId();
        }

        if (scrollId != null) {
            System.out.println("clear " + scrollId);
            String finalScrollId = scrollId;
            client.clearScroll(cs -> cs.scrollId(finalScrollId));
        }
    }

    @Test
    public void testQuery() {

        MatchQuery.Builder mb = new MatchQuery.Builder();
        mb.field("agent").query("xxx");
        Query query = Query.of(qb -> qb.match(mb.build()));

        Instant now = Instant.now();
        Instant threeDaysAgo = now.minus(Duration.ofDays(3));
        long threeDaysAgoMillis = threeDaysAgo.toEpochMilli();
        Query rangequery = RangeQuery.of(r -> r.field("@timestamp").lte(JsonData.of(now.toEpochMilli())).gte(JsonData.of(threeDaysAgoMillis)))._toQuery();
        Query boolquery = Query.of(qb -> qb.bool(b -> b.must(query).must(rangequery)));
        System.out.println(boolquery);
        
        MatchPhraseQuery.Builder builder = new MatchPhraseQuery.Builder();
        builder.field("agent").query("xxx");
        Query phraseQuery = Query.of(qb -> qb.matchPhrase(builder.build()));
        System.out.println(phraseQuery);
        
    }

    @Test
    public void testSearch() throws IOException {

        SortOptions.Builder sb = new SortOptions.Builder();
        SortOptions sortOptions = sb.field(fs -> fs.field("@timestamp").order(SortOrder.Desc)).build();
        final SearchRequest.Builder searchReq = new SearchRequest.Builder().allowPartialSearchResults(false)
            .index(Collections.singletonList(indexName)).size(10).sort(sortOptions).source(sc -> sc.fetch(true))
            .trackTotalHits(tr -> tr.enabled(true)).query(q -> q.matchAll(new MatchAllQuery.Builder().build()));
        SearchResponse<LogEntry> searchResponse = client.search(searchReq.build(), LogEntry.class);
        System.out.println(searchResponse.hits().total());
        for (Hit<LogEntry> hit : searchResponse.hits().hits()) {
            System.out.println(mapper.writeValueAsString(hit.source()));
        }
    }

    @Test
    public void testTermsAgg() throws IOException {
        MatchAllQuery.Builder mb = new MatchAllQuery.Builder();
        Query query = Query.of(qb -> qb.matchAll(mb.build()));
        final SearchRequest.Builder searchReq = new SearchRequest.Builder().allowPartialSearchResults(false)
            .index(Collections.singletonList(indexName)).query(query).size(0).trackTotalHits(tr -> tr.enabled(true))
            .aggregations("hostname_group", a -> a.terms(t -> t.field("agent.hostname.keyword")));

        SearchResponse<LogEntry> searchResponse = client.search(searchReq.build(), LogEntry.class);
        System.out.println(searchResponse.hits().total());
        List<StringTermsBucket> buckets = searchResponse.aggregations().get("hostname_group").sterms().buckets().array();
        for (StringTermsBucket bucket : buckets) {
            System.out.println(bucket.docCount() + " terms under " + bucket.key().stringValue());
        }
    }

    @Test
    public void testSingleDocument() throws IOException {
        Product product = new Product("bk-1", "City bike", 123.0);
        IndexRequest<Product> request = IndexRequest.of(i -> i.index("products").id(product.getSku()).document(product));

        IndexResponse response = client.index(request);

        System.out.println("Indexed with version " + response.version());
    }

    @Test
    public void testSingleDocumentDSLAsync() throws Exception {
        Product product = new Product("bk-1", "City bike", 123.0);

        EasysearchAsyncClient asyncClient = SampleClient.createAsyncClient();
        CompletableFuture<IndexResponse> future = asyncClient.index(i -> i.index("products")
            .id(product.getSku()).document(product)).whenComplete((response, exception) -> {
            if (exception != null) {
                System.err.println("Failed to index " + exception);
            } else {
                System.out.println("Indexed with version " + response.version());
            }
        });
        IndexResponse response = future.get();
        System.out.println("Async Indexed with version " + response.version());
    }
}

```

