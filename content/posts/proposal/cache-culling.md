---
title: "Cache Culling"
date: 2022-02-17T21:07:03+08:00
draft: true
toc: true
tags: 
  - proposal
---

今天解锁新的分组，**proposal** ，记一下实际方案的 trade offf。

缓存删除问题

缓存的作用就是为了快，有做数据库用的，有的就是单纯为了提高访问速度防止高频次访问拖垮基础服务，另外的就是分布式锁、消息队列。。。

单就做缓存的话，为了提高服务的质量，需要保证缓存的数据一直是最新的。

剔除缓存中的旧数据，更新为新的数据时，多线程会导致**数据的不一致问题**。



- 先删除缓存再更新数据库   缓存可能一直是旧数据，可能被其他线程又塞进去了
- 先更新数据库再删除缓存，短时间内存在脏数据
- 延迟双删，超时时间内数据不一致，延迟的时间需要根据实际业务场景来决定
  - 先删除缓存
  - 写数据库
  - 休眠指定时间
  - 再次删除缓存
    - 删除失败，一直是脏数据
    - 将需要删除的 key 发送到 MQ
    - MQ 对删除操作重试，直到成功
- 基于MQ删除缓存，binlog -> canal -> Kafka 顺序消息 -> MQ 删除重试



当系统中引入了不可靠的中间件，不仅会使的系统整体的可用性打折扣，还需要侵入逻辑代码去保证。

对整体的系统规模进行估算测量后，选择合适的方案。
















