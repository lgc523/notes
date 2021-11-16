---
title: "Prometheus"
date: 2021-08-21T11:26:29+08:00
draft: true
toc: true
images:
tags: 
  - go
---

Prometheus 由 SoundCloud 用Go语言编写并开源的监控告警系统，自带时序数据库，采用 Pull 方式获取监控信息，提供了多维度的数据模型和灵活的查询接口。

不仅可以通过静态文件配置监控对象，还支持自动发现机制，能够通过 K8s、etcd、Consul、DNS 等多种方式动态获取监控对象。

单机可以采集数百个节点的监控数据，每秒可以采集一千万个指标，还支持远端存储。

## 监控系统





## 架构

### Pull

Prometheus 通过 HTTP 周期性的抓取被监控组件的状态，任意组件只要提供对应的 HTTP 接口并且符合 Prometheus 定义的数据格式，就可以接入 Prometheus 监控

通过 Pull 的方式调用被监控数据获取监控数据的方式，能够自动进行上游监控和水平监控，配置更少，更容易拓展、实现高可用、降低耦合，可以避免推送系统中容易出现的推送数据失败导致瘫痪的问题，采集端无感监控系统的存在，独立监控系统之外，增强了监控系统整体的可控性，但是需要管理多个 exporter。

### 获取监控对象

- 通过配置文件静态配置
- 支持 Zookeeper、Consul、Kubernetes 动态发现

### 存储

Storage 通过一定规则清理和整理数据，把得到的结果存储到新的时间序列中。

- 本地存储（自带时间序数据库，大量数据有瓶颈）

- 远端存储，通过中间层的适配器转化实现 Prometheus 存储的 remote write ,remote read 接口接入

   支持常用的 Kakfa、OpenTSDB、InfluxDB、ElasticSearch

### 查询

Prometheus 通过 PromQL 和其他 API 可视化展示收集的数据，支持 Grafana、自带的 PromDash 、自身提供的模版引擎、HTTP API

### PushGateway

对于某些系统通过 Push 方式实现的推送，Prometheus 提供了对 PushGateway 的支持，这些系统主动过推送 metrics  到  PushGateway，Prometheus 定时去 Gateway 上抓取数据。

### AlertManager

AltertManager 是独立于 Prometheus 的一个组件，在触发了预先设置的高级规则后，Prometheus 将会推送告警信息到 AlertManager

AlertManager 告警方式，可以通过邮件、slack、或者 dingTalk 等途径推送。

AlertManager 支持高可用部署，为了解决多个 AlertManager 重复告警问题，引入了 Gossip，在多个 AlertManager 之间同步告警信息

## 其他开源监控工具

RDD (Round Robin Database 环形数据库)

RDD 存储： 将整个数据存储空间构成圆环，指针指向最新的数据位置并且随着数据读写移动，如果此时没有获取监控数据，RDD 会使用默认的 unknown 填充，保证数据对齐，每个数据库文件都以 .rdd 结尾，大小是固定的。

- Zabbix （公司在用。。。）,1998 年基于c + php + 关系型数据库 实现，性能不行，支持主动推送、被动拉取
- Nagios   1999 年c 语言实现，老牌监控，主要针对主机、网络监控，比较稳定，通过 plugin 采集各种监控数据
- Open-Falcon 小米开源 Go 语言实现，RDD 存储，加入一致性 Hash 进行数据分片，可以对接 OpenTSDB，容器监控支持力度有限

## 安装配置

[下载](https://prometheus.io/download/)对应 OS 的压缩包，

## metric

Prometheus metric 统一定义为，涉及 指标名称和标签两部分

```
<metric name> {<lable name>=<label value>,...}
```

- metric name

  说明 metric 含义，必须由字面量、数值下划线或者冒号组成，符合正则表达式 { [[a-zA-Z_:]][][a-zA-Z_:]* }

  其中的冒号指标不能用于exporter。

- metric label

  体现指标的维度特征，用于过滤、聚合，通过 label_name 和 label_value 这种键值对形成多种维度。

  有些 label  是以 “—“ 开头的，这些 label 是在 Prometheus 内部使用。

### metric category

**长尾效应**：统计学术语，主要描述 极低值(价值、数值)的个体数量占了总体的绝大多数。



Prometheus 指标分为 counter (计数器)、Gauge (仪表盘)、Histogram (直方图)、Summary (摘要)

- counter

  只增不减，eg: 机器启动时间、HTTP 访问量

- Gauge

  指标的实时变化情况，可增可减，eg: CPU、内存的使用量、网络IO 大小，大部分数据都是 Gauge 类型的。

- Summary

  高级指标，采样点分位图的到数据的分布情况，用于凸显数据的分布情况。

  如果想要了解某个时间段内请求的响应时间，通常使用平均响应时间(无法体现数据的长尾效应)，可以使用 Summary/Histogram

- Histogram

  反应某个区间内的样本个数，通过 {le="上边界"} 指定这个范围内的样本数

### 数据样本

Prometheus 采集的数据样本都是以时间序列保存的，每个样本都由三部分组成，指标、样本值、时间戳

样本值(64位浮点数) 和时间戳(ms) 的组合代表这个时间点采集到的监控数值，这些时序数据首先被保存子啊内存中，然后被批量刷新到磁盘。

### 数据采集

Pushtgateway 组件接受客户端发送过来的数据，按照 Job 和 instance 两个层级进行组织，支持数据的追加和删除，为防止数据丢失，还支持本地存储

- 实时性

  Push 实时性相对较好，可以将采集数据立即上报到监控中心

  Pull 方式通常进行周期性采集，采集时间为 30s 或更长时间，实时性要求非常高的的监控可以采用 Push 方式。

- 状态保存

  Push 方式通常在采集完成后立即上报，本地不会保存此采集数据，Agent 本身是没有状态的，Master 需要维护各种 Agent状态

  Pull 方式，Agent 本身有一定的数据存储能力，Master 只负责简单的数据拉取，而且本身可以做到无状态。

- 控制能力

  Push 方式，控制方是 Agent ,Agent 上报的数据决定了上报的周期和内容

  Pull 方式，Master 更加主动，控制采集的内容和频率

- 配置的复杂性

  Push 方式，每个 Agent 需要配置 Master 地址

  Pull 方式，通常通过批量配置或者自动发现来获取所有采集点，相对简单，并且可以做到和 Agent 充分解耦，Agent 不用感知 Master 存在。

### 服务发现

- 静态文件配置

  适用于有固定的监控环境，只需要配置监控对象的ip+port 就行，后面进行周期性的调度就可以。
