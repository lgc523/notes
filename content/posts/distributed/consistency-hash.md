---
title: "Consistency Hash"
date: 2022-01-07T13:18:09+08:00
draft: true
toc: true
images:
tags: 
  - distributed
---

这几天在看 redis 看了不少哈希，昨天晚上又看之前的分布式的东西，看到分布式高可靠的负载均衡，里面也有哈希来做负载均衡，但是都没有解决有状态的，根据请求资源消耗来做负载的，之前总听说 go-zero 有一些自适应这，自适应那的东西，正好一块记录一下，后面有空用用 go-zero，目前对 go-zero 的了解只有有一些 tool，redis 只有 0 库，。。。

看了一下 go-zero 自适应负载的大概算法，https://learnku.com/articles/60059，上厕所想到了 TCP 的 RTO ，不过这个是在分布式环境下的，TCP RTO 是针对一个连接的。

