---
title: "Latency、DT、Approximate Value"
date: 2022-01-04T20:42:23+08:00
draft: true
toc: true
images:
tags: 
  - engineering
---

2022 元旦看的东西，忘记了是什么东西，大概是关于系统时间的一些数字，之前看过一些数字，暂且先记录一下。

## latency

[latency number](https://gist.github.com/jboner/2841832)

```

Latency Comparison Numbers (~2012)
----------------------------------
L1 cache reference                           0.5 ns
Branch mispredict                            5   ns
L2 cache reference                           7   ns                      14x L1 cache
Mutex lock/unlock                           25   ns
Main memory reference                      100   ns                      20x L2 cache, 200x L1 cache
Compress 1K bytes with Zippy             3,000   ns        3 us
Send 1K bytes over 1 Gbps network       10,000   ns       10 us
Read 4K randomly from SSD*             150,000   ns      150 us          ~1GB/sec SSD
Read 1 MB sequentially from memory     250,000   ns      250 us
Round trip within same datacenter      500,000   ns      500 us
Read 1 MB sequentially from SSD*     1,000,000   ns    1,000 us    1 ms  ~1GB/sec SSD, 4X memory
Disk seek                           10,000,000   ns   10,000 us   10 ms  20x datacenter roundtrip
Read 1 MB sequentially from disk    20,000,000   ns   20,000 us   20 ms  80x memory, 20X SSD
Send packet CA->Netherlands->CA    150,000,000   ns  150,000 us  150 ms

Notes
-----
1 ns = 10^-9 seconds
1 us = 10^-6 seconds = 1,000 ns
1 ms = 10^-3 seconds = 1,000 us = 1,000,000 ns
```

[Challenges in Building Large-Scale Information Retrieval Systems](http://videolectures.net/wsdm09_dean_cblirs/)

## DownTime 9{2,6}

### 可靠性 MTBF

Mean Time Between Failure，平均故障间隔时间，失败率 = 1/MTBF，单位 1 FITs=10^-9(1/h)。

### 可维修性 MTTR

Mean Time To Recover，平均故障修复时间，修复率 = 1/TMTTR。

### 可用度

$$
可用度 A = MTBF/(MTBF + MTTR)
$$



### Down Time

**停机时间 DT = (1-A) * 8760 * 60 (分钟/年)**



| Availability % | DT per day     | DT per year |
| -------------- | -------------- | ----------- |
| 99%            | 14.40 min      | 3.65 day    |
| 99.9%          | 1.44 min       | 8.77 hour   |
| 99.99%         | 8.64 sec       | 52.60 min   |
| 99.999%        | 864.00 millsec | 5.26 min    |
| 99.9999%       | 86.40 millsec  | 31.56 sec   |

- **网站平均响应时间在 500 ms 左右，理想时间在 [200-300]ms。**

- **平均 QPS 日平均请求除以 4w，pk QPS = [2,4] 平均 QPS。**

## 000

**1000(quadrillion) ,000(trillion) ,000(billion), 000(million), 000(thousand)**

2^10 = 1024 = 1KB ≈ 1,000 1 Thousand

2^20 = 1,048,576 = 1MB ≈ 1,000,000 = 1 Million

2^30 = 1,073,741,824 = 1GB ≈ 1,000,000,000 = 1 Billion

2^40 = 1,099,511,627,776 = 1TB ≈ 1,000,000,000,000 = 1 Trillion

2^50 = 1.125899906842624E15 = 1PB ≈ 1Quadrillion

## KB |  KIB

- KiB是kilo binary byte的缩写，是千位二进制字节。
- KB 是kilobyte的缩写，是千字节。

