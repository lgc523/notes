---
title: "Mysql Cli"
date: 2022-02-17T15:13:04+08:00
draft: true
toc: true
images:
tags: 
  - Mysql
---

今天通过 show slave status 看 slave 状态，发现了一些命令，记一下，后面记一下常用的命令。

一开始执行 ``show salve status\G;`` 发现有 ``warning`` 

``show warning``  显示 slave 命令 过时，将支持 ``show replica status``, 并且 ``\G`` 不用加 ``；``。

![salve-status](https://s2.loli.net/2022/02/17/XSb34uWI6t7TwaF.png)



