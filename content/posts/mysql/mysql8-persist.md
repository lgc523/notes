---
title: "MySQL8 Persist"
date: 2021-10-07T09:18:59+08:00
draft: true
toc: true
images:
tags: 
  - MySQL
---

MySQL8添加了支持全局参数修改持久化功能，通过命令可以将参数持久化到文件 **mysqld-auto.cnf (JSON格式)**.

mysql.cnf 和 mysqld-auto.cnf **同时存在，后者优先级更高。**

命令 **set persist xx=yy**;

- **set persist 可以修改内存中变量的值**
- **set persist_only 不会修改内存中变量的值**

二者都会将修改后的值持久化到 mysqld-auto.cnf，默认文件在 **/var/lib/mysql** 下。

即使修改的值没有变化，还是会持久化，可以通过 **set persist xx=default**; 设置默认值

**移除所有全局参数 reset persist; / 删除 mysqld-auto.cnf 文件后重启**



<img src="http://guangchang.tech:5023/images/mysql/mysql8_persist.png?name=mysql8_persist.png&download=1" alt="persist" style="zoom:33%;" />

