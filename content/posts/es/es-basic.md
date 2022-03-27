---
title: "ES Basic"
date: 2022-01-30T20:43:01+08:00
draft: true
toc: true
tags: 
  - es
---

## 节点

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
  - Id 不存在创建新的 doc，否则删除现有的文档
- create
  - id 存在会失败
- read
- update
  - 文档必须存在，更新只会对相应字段做增量修改
- Delete



## Create

``PUT index_name/_create/{id}``

```
指定 文档id
```

``POST /index_name/_doc/``

```
自动生成id
```

