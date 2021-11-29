---
title: "es Install"
date: 2021-11-29T22:25:44+08:00
draft: true
toc: true
images:
tags: 
  - Elasticsearch
---

## Lucene

- 基于 Java 语言开发的搜索引擎类库
- 创建于1999年，2005 年成为 Apache 顶级开源项目
- 高性能、易拓展
- 局限性
  - 只能基于 Java 语言开发
  - 类库的接口学习曲线陡峭
  - 原生并不支持水平拓展

## Elasticsearch

- 2004 年 Shay Bacon 基于 Lucene 开发了 Compass
- 2010 年 重写 Compass，取名 Elasticsearch
  - 支持分布式，可水平拓展
  - 降低全文检索的学习曲线，可以被任何编程语言调用 https://www.elastic.co/guide/en/elasticsearch/client/index.html
  - RESTful ，JDBC & ODBC
- 海量数据的分户式存储以及集群管理
  - 服务与数据的高可用，水平拓展
- 近实时搜索，性能卓越
  - 结构化/全文/地址位置/自动完成
- 海量数据的近实时分析
  - 聚合功能

## version and feat

- 0.4 2010.2
- 1.0 2014.1
- 2.0 2015.10
- 5.0 2016.10
  - Lucenn 6.x，性能提升，移除同一文档并发更新的竞争锁
  - Instant aggregation ，支持分片上聚合的缓存
  - 新增了 Profile API
- 6.0 2017.10
  - Lucenn 7.x
  - 跨集群复制 CCR、索引生命周期管理、SQL 支持
  - 更友好快速的版本升级和数据迁移
  - 有效存储稀疏字段的新方法，降低了存储成本
  - 在索引时进行排序，可加快排序的查询性能
- 7.0 2019.4
  - Lucene 8.0
  - 正式废除单个索引下多 Type 的支持
  - 7.1 开始 Security 功能免费使用
  - ECK Elasticsearch Operator on Kubernetes
  - New Cluster coordination
  - Feature-Complete High Level REST Client
  - Script Score Query
  - 默认的 Primary Shard 数从 5 改为 1，避免 Over Sharding
  - 更快的 Top K

## Ecosphere

解决方案 ``搜索`` ``日志分析`` ``指标分析`` ``安全分析``

- 开源
  - 可视化 Kibana
  - 存储计算 Elasticsearch
  - 数据抓取 Logstash Beat
- 商业
  - X-Pack  ``安全`` ``告警`` ``监控`` ``图查询`` ``机器学习`` 

## Logstash

``数据处理管道``，诞生于 2009，2013 年被 Elasticserach 收购。

最初用来做日志的采集与处理，服务端数据处理通道，支持从不同来源采集数据，转换数据，并将数据发送到不同的存储库中。

### feat

- 实时解析和转换数据
  - 将 IP 地址破译出地理坐标
  - 将 PII 数据匿名化，完全排除敏感字段
- 可拓展
  - 200 多插件 （日志/数据库/Arcsigh/Netflow）
- 可靠性安全性
  - Logstash 会通过持久化队列来保证至少将运行中的事件送达一次
  - 数据传输加密
- 监控

## kibana

``Kiwifruit + Bannan``

基于 Logstash 的工具，2013 免费被 Elasticsearch 收购。

## BEATS

轻量的数据采集器，GO语言开发。

- FileBeat         	Real-time insight into log data
- Packetbeat       Analyze network packet data
- Winlogbeat      Analyze Windows event logs
- Mericbeat        Ship and analyze metrics
- Heartbeat        Ping your infrastructure
- Auditbeat        Send audit data to Elasticsearch
- Functionbeat  Ship cloud data with serveless infrastructure
- Journalbeat     Analyze journals logs

## ELK  APPLY

- 网站搜索/垂直搜索/代码搜索
- 日志管理与分析/安全指标监控/应用性能监控/WEB抓取舆情分
- 日志搜集 -> 格式化分析 -> 全文检索 -> 风险告警

```
+-------------------------------------------------------------------------------------------------------+
|                                                                                                       |
|                                                                                                       |
| +----------------------+                                         Grafana                              |
| | Porgraming Language  | ----->                                                                       |
| +----------------------+                                            |                                 |
|                                                                     |                                 |
|                                                                     |                                 |
|                             redis                                   v                                 |
|                                                                                                       |
|              beats ----->   kafka    ----->  logstash  -----> elasticsearch  <----- kibana            |
|                                                                                                       |
|                            rabbitmq                                                                   |
|                                                                                                       |
|       --------------------------------------------------------------------------------------------    |
|              Data            Buffering        Data                Indexing &          Analysis &      |
|              Collection                       Aggregation         storage           visualization     |
|                                               & Processing                                            |
|                                                                                                       |
+-------------------------------------------------------------------------------------------------------+
```

## directory structure

- bin			脚本文件（启停/插件/统计）
- config      elastic search.yml 集群配置文件，user、role based 相关配置
- JDK           7.x 内置 java 环境
- data         path.data 数据文件
- lib             java 类库
- logs          path.log 日志文件
- modules  所有模块
- plugins     已安装插件 

## JVM OPTS

``config/jvm.options``

7.1 默认设置 1GB

Xmx 不要超过机器内存的 50 %，内存不要超过 30G

https://www.elastic.co/cn/blog/a-heap-of-trouble

- Jvm.options -Xms = -Xmx =2g
- Elastic search.yml 
  - node.name 
  - discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes 其中一个指定为 node.name
  - network.host: 0.0.0.0
- /etc/sysctl.conf 
  - vm.max_map_count 调大 655360
  - sysctl -p

## cli

- bin/elasticsearch-plugin list
- bin/elasticsearch-plugin install analysis-icu 装完插件重启生效，不知道有没有reload

## RESTful

1. _cat/plugin
2. _cat/nodes



## Q

- ssh abrt  timeout

## feat

查看外网ip

- *curl cip.cc*
- *curl myip.ipip.net*
- curl ip.sb
- curl inet-ip.info
- curl ipinfo.io (json)

之前公司运维有一个命令，没看到，c



