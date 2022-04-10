---
title: "ES Basic"
date: 2022-01-30T20:43:01+08:00
draft: true
toc: true
tags: 
  - es
---

## 节点

```
GET /{indexName}/_search_shards  
"roles" : [
        "data",
        "data_cold",
        "data_content",
        "data_frozen",
        "data_hot",
        "data_warm",
        "ingest",
        "master",
        "ml",
        "remote_cluster_client",
        "transform"
      ]
```



- 每个节点启动，默认是 master eligible 节点，可以设置 node.master:false 禁止
- master-eligible 节点可以参加选主流程，成为 master 节点
- 第一个节点启动时候，会将自己选举成 master 节点
- 每个节点都保存了集群的状态，只有 master 节点才能修改集群的状态信息
  - node.master: true
  - 集群状态 cluster state 维护了一个集群中的必要信息
    - 所有的节点信息
    - 所有的索引和其相关的 mapping 与 setting 信息
    - 分片的路由信息
  - 任意节点都能修改信息将会导致数据不一致
  - ？集中式中心架构？
- Data node
  - node.data: true
  - 保存数据的节点，负责保存分片数据
- Coordinating node 协调节点
  - 请求会跨越多个节点的时候（bulk_indexing，search）
  - Coordinating 节点负责接收 client 的请求，将请求分发到合适的节点，最终把结果汇集到一起，**scatter-gather**
    - **scatter阶段**， 转发请求给含有相关数据的节点
    - Data 节点在本地执行请求后把结果返回给 coordinating 节点
    - **gather 阶段**，Coordinating 节点汇总结果成一个单一的全局结果集
  - **每个节点默认都起到了 Corrdinating node 的职责**
- Hot & Warm node
  - 不同硬件配置的 data node 实现冷热数据，降低成本
- Machine Learning Node
  - 负责跑机器学习的 Job，用来做异常检测
- ingest node 提取节点
  - node.ingest: true
  - 可以通过 ingest pipeline 对文档执行预处理操作，以便在索引文档之前对文档进行转换或者增强
- Client node
  - 客户端节点
    - node.master : false
    - node.data: false
  - 节点只能处理路由请求，处理搜索，分发索引操作等，表现为智能负载平衡器
  - 主节点必须等待每一个节点集群状态的更新确认，客户端节点太多会成为负担。

## shard

```
GET /_cat/shards?v
```



### Primary shard

主分片，解决数据水平拓展的问题，通过主分片，可以将数据分布到集群内的所有节点之上

- 一个分片是一个运行的 Lucene 的实例
- 主分片数载索引创建时指定，后续不允许修改，除非 reindex

需要提前做好容量规划

- 分片设置过小
  - 后续无法增加节点实现水平拓展
  - 单个分片的数据量太大，导致数据重新分配耗时
- 分片设置过大，7.0 开始，默认主分片设置为1，解决了 over-sharding 的问题
  - 影响搜索结果的相关性打分，影响统计结果的准确性
  - 单个节点上过多的分片，会导致资源浪费，同时影响性能

### Replica shard

副本，解决数据高可用问题，分片是主分片的拷贝

- 副本分片数，可以动态调整
- 增加福本数可以在一定程度上提高读取的吞吐

## Cluster health

``GET /_cluster/health``

```apl
{
  "cluster_name" : "spider-es-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 26,
  "active_shards" : 52,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

- Green 主分片和副本都正常分配
- Yellow 主分片全部正常分配，有副本分片未能正常分配
- Red 有主分片未能分配
  - 磁盘不够时

```
GET /_cat/nodes
GET /_cat/shards
```

## CRUD

- index
  - **Id 不存在创建新的 doc，否则删除现有的文档，新的文档被索引，版本信息 +1**
- create
  - id 存在会失败
- read
- update
  - 文档必须存在，更新只会对相应字段做增量修改
- Delete



### Create

``PUT index_name/_create/{id}``

```
指定 文档id
```

``POST /index_name/_doc/``

```
自动生成id
```

### GET

```
GET /users/_doc/2
{
  "_index" : "users",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 1,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "firstName" : "Jack",
    "lastName" : "Johnson",
    "tags" : [
      "guitar",
      "stateboard"
    ]
  }
}
```

同一个 ID 的文档，即使被删除，version 号也会不断增加。

### Update

- 不会删除原来的文档，实现真正的数据更新
- Post 方法，Payload 需要包含在 doc 中

```
POST /users/_update/2
{
  "doc":{
    "firstName":"spider"
  }
}
```

### index

```
PUT users/_doc/{id}
先删除，再更新版本号
```

## Bulk

支持在一次 API 调用中，对不同的索引进行操作。

支持四种类型操作

- index
- create
- update
- delete

单个操作失败不会影响其他操作，返回每个操作执行的结果。

```
POST [/{index_name}/]_bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{"delete":{"_index":"testa","_id":"1"}}
{"delete":{"_index":"test","_id":"2"}}
```

## mget

根据 doc id查询

```
GET [{index_name}]/_mget
{
  "docs":[
    {
      "_index":"test",
      "_id":1
    },
    {
      "_index":"movies",
      "_id":1
    },
     {
      "_index":"movies",
      "_id":29876
    }
    ]
}
```

## msearch

根据查询条件查询

```
POST kibana_sample_data_ecommerce/_msearch
{}
{"query":{"match_all":{}},"size":1}
{"index":"kibana_sample_data_ecommerce"}
{"query":{"match_all":{}},"size":1}
{"index":"test"}
{"query":{"match_all":{}},"size":2}
```

## Err status_code

| 问题         | 原因               |
| ------------ | ------------------ |
| 无法连接     | 网络故障/集群down  |
| 连接无法关闭 | 网络故障活节点出错 |
| 429          | 集群过于繁忙       |
| 4xx          | 请求体格式有错     |
| 500          | 集群内部错误       |

## info

```

#查看索引相关的信息
GET kibana_sample_data_ecommerce
#查看索引的文档个数
GET kibana_sample_data_ecommerce/_count
GET test
POST kibana_sample_data_ecommerce/_search
{
}
GET /_cat/indices/kibana*?v&s=index
GET /_cat/indices?v&health=green
GET /_cat/indices?v&s=docs.count:desc
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt
GET /_cat/indices?v&h=i,tm&s=tm:desc
GET /_cat/nodes?v
GET /_cat/nodes?v&h=id,ip,port,v,m
GET /_cat/shards
GET /_cat/indices
GET /_cluster/health?level=shards
```

## analyzer

Analysis 文本分析是把全文本转换一系列单词 term/token 的过程。

Analysis 通过 analyzer 来实现的，可以使用内置的分析器或者按需定制化分析器。

数据写入和匹配 Query 语句时需要用相同的分析器对查询语句进行分析。

### 组成

- **character filters**
  - 针对原始文本处理，eg 去除 html 标签
- **tokenizer**
  - 按照规则切分单词
- **token filter**
  - 将切分的单词进行加工，大小写转换，删除 stop words



![analyzer](https://s2.loli.net/2022/04/04/2uH6VjT5tqAlSEL.png)

### 内置分词器

- Standard Analyzer 默认粉刺起，按词切分，小些处理
- Simple Analyzer     按照非字母切分，符号被过滤，小写处理
- Stop Analyzer 小写处理，停用词过滤（the ,a , is）
- Whitespace Analyzer 按照空格切分，不转小写
- Keyword Analyzer 不分词，直接将输入当作输出
- Patter Analyzer 正则表达式，默认 \W+ （非字符分隔）
- Language 提供了30多种常见语言的分词器
- Customer Analyzer 自定义分词器

### _analyzer API

- 直接指定 Analyzer 进行测试

  ```
  GET /_analyze
  {
    "analyzer": "standard",
    "text":"Master Elasticsearch,elasticsearch in Action"
  }
  ```

- 指定索引的字段进行测试

  ```
  POST movies/_analyze
  {
    "field": "title",
    "text": ["Mastering Elasticsearch"]
  }
  ```

- 自定义分词器进行测试

  ```
  POST /_analyze
  {
    "tokenizer": "standard",
    "filter": ["lowercase"],
    "text":"Mastering Elasticsearch"
  }
  ```

### standard analyzer

- 默认分词器
- 按词切分
- 小写处理

![standard_analyzer](https://s2.loli.net/2022/04/04/QYpcJXr7fmjvL5G.png)

```
GET /_analyze
{
  "analyzer": "standard",
  "text": ["2 running Quick brown-foxes leap over lazy dogs in the summer evening"]
}
只会对词进行切分，不过过滤停用词 in the
```

### simple analyzer

- 非字母切分，非字母的都被去除
- 小写处理

```
GET /_analyze
{
  "analyzer": "simple",
  "text": ["2 running Quick brown-foxes leap over lazy dogs in the summer evening"]
}
brown
foxes
```

### whitespace

- 按照空格切分

```
GET /_analyze
{
  "analyzer": "whitespace",
  "text": ["2 running Quick brown-foxes leap over lazy dogs in the summer evening"]
}
12 个 token
```

### stop analyzer

- 非字母切分，非字母的都被去除
- 小写处理
- Stop  filter
  - 去除 the a is 等修饰性词语

```
GET /_analyze
{
  "analyzer": "stop",
  "text": ["2 running Quick brown-foxes leap over lazy dogs in the summer evening"]
}
2 也没有了，
```

### Keyword analyzer

```
GET /_analyze
{
  "analyzer": "keyword",
  "text": ["2 running Quick brown-foxes leap over lazy dogs in the summer evening"]
}
不分词
{
  "tokens" : [
    {
      "token" : "2 running Quick brown-foxes leap over lazy dogs in the summer evening",
      "start_offset" : 0,
      "end_offset" : 69,
      "type" : "word",
      "position" : 0
    }
  ]
}
```

### Pattern analyzer

### ICU Analyzer

- Elastic search-plugin install analysis-icu
- 提供了 unicode 支持，更好的支持亚洲语言

## 倒排索引



## Search api

- URL Search

  - GET 方法，在 url 中使用查询参数
  - 使用 'q' 指定查询字符串
  - "query string syntax" KV 键值对

  ```
  cu http://localhost:9200/kibana_sample_data_ecommerce/_search?q=customer_first_name:Eddie | json
  ```

  | 语法                   | 范围                                     |
  | ---------------------- | ---------------------------------------- |
  | /_search               | 集群上所有索引                           |
  | /index1/_search        | index1                                   |
  | /index1.index2/_search | index1.index2，一条语句再多个index上执行 |
  | /index*/_search        | Index 开头的索引                         |

  

- Request Body Search

  - 基于 JSON 格式的 Query Domain Specific Language

  ```
  cu -XGET/POST http://localhost:9200/kibana_sample_data_ecommerce/_search -H "Content-type:application/json" -d '{"query":
  			{
  				"match_all":{}
  			}
  		 }' | json
  ```

Relevance

衡量相关性

Information retrieval

- precision 查准率，尽可能返回较少的无关文档
- recall 查全率，尽量返回较多的相关文档
- ranking 是否能够按照相关度进行排序

## URI Search

- q 指定查询语句，query string syntax
- df 默认字段，不指定时会对所有字段进行查询
- sort 排序,from, size 用于分页
- profile 可以查看查询时如何被执行的

### query string syntax



