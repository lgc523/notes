---
title: "Master Slave 主从复制配置"
date: 2021-09-28T23:12:07+08:00
draft: true
toc: true
images:
tags: 
  - mysql
---

## 主从复制

从结点启动 io 线程监测主结点 bin log 变化，读到 relay log 里，SQL 线程负责从 realy log 日志里面读出 binlog 内容，更新到  slave 数据库中，保证主从数据库数据一致。

## 一主一从

**master 宕机 只能读 不能写**

### 主/从节点

```
vi /etc/my.cnf
添加
server-id=1
log-bin=mysql-bin
#写binlog 数据库
binlog-do-db=gc 
#不写binglog
binlog-ignore-db=mysql 
binlog-ignore-db=information_schema
#行模式
binlog-format=row 
slave-skip-errors=all
```

```
server-id=3
log-bin=mysql-bin
#从节点作为下属从节点的主节点
log-slave-updates
replicate-do-db=gc
replicate-ignore-db=mysql 
replicate-ignore-db=information_schema
```

### 从节点

```
#	账号权限 【REPLICATION SLAVE, REPLICATION CLIENT】
# master 上 show master status
# master_connect_retry 重试的时间间隔，单位是秒，默认是60秒
change master to
     master_host='xxx',
     master_user='xxx',
     master_password='xxx',
     master_port=xxx,
     master_log_file='mysql-bin.000001', 
     master_log_pos=502,
     master_connect_retry=30;
start slave;
show slave status\G;
成功状态
  【Slave_IO_Running: Yes】
  【Slave_SQL_Running: Yes】
```

## 异步复制
