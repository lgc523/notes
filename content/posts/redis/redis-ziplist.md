---
title: "Redis Ziplist"
date: 2022-01-11T01:03:09+08:00
draft: true
toc: true
images:
tags: 
  - redis
---

Ziplist 底层实现更紧凑，节省内存，没有指针的使用，基于 ziplist 实现了 LIst、Hash、Sorted Set 。

## Ziplist 结构

- zlbytes 列表长度 4B ，记录整个列表占用的内存字节数，内存重分配或则会计算 zlend 时使用。
- zltail     列表尾的偏移量
- zllen  entry 个数，uint16_t，< 65535 时，就是压缩列表的数量， **= 65535 时，真实数量需要遍历计算**。
- 中间是entry     可以保存一个字节数组或者整数值
  - prev_len   前一个 entry 的长度 1B/5B，1B=上一个entry.len < 254B，5B=len+zlend，**应用：从表尾遍历到表头**
  - len            自身长度 4B
  - encoding 编码方式 1B
  - content   实际数据
- zlend 列表结束 1B（default=255/0xFF，表示列表结束）

中间 entry 相邻存放，不需要用额外的指针进行链接，可以节省指针占用的空间，同时也减少了在全局哈希表的k、v占用，v(n)。

### 压缩列表节点的构成

每个列表节点可以保存一个字节数组或者整数值，分别对应的长度

- 字节数组
  - <= 63，2^6-1 B
  - <=16383，2^14-1 B
  - <=4 294 967 295， 2^32-1
- 整数值
  - uint 4bit 0-12
  - 1B int
  - 3B int
  - int16_t
  - Int32_t
  - Int64_t

### encoding

记录节点的 content 属性保存的数据类型以及长度。

- 1B/2B/5B，值的最高位为 00，01，10 表示是字节数组编码，数组的长度由编码除去最高两位之后的其他位记录。
- 1B，值的最高位以11开头的表示是整数编码，整数值的类型和长度由编码除去最高两位之后的其他位记录。

### Cascade update

Entry prev_len 存储的是前一个 entry 的长度，最坏的情况，在**表头插入一个长度大于 254 的 entry**，导致后面所有的 entry 内存一直重分配到最后。

如果**删除列表中 big_entry 后面的 small_entry** ，最坏的情况易导致后面的所有的 entry 级联更新。

**最坏的情况需要列表中存在连续并且长度在 250-253 之间的并且数量比较多的节点，才会对性能造成影响。**

当列表键包含少量列表项时，Redis 会使用压缩列表来做列表键的底层实现， **object encoding mylist** 可以观察底层实现，benchmark 写入的 10000 个 value 到 mylist，底层就没用 ziplist。
