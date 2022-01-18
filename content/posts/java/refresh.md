---
title: "Refresh File"
date: 2022-01-18T22:06:34+08:00
draft: true
toc: true
images:
tags: 
  - java
---

早就想写一下动态刷新方面的东西，主要是项目中的配置文件更新，最近也准备看一下 spring-cloud 方面的东西，大概是 slf4j，nacos，nio，air（golang）动态更新的东西，这都是在刚刚洗澡的时候想起来的，大概的原理都知道，只是没空写。。。

其实想起来的是：项目中引用的外部 sdk，有一些方法的签名涉及到文件的路径，由于springboot maven plugin 打包的原因，导致无法获取到文件路径，现在的方案是通过 io 在 spring 启动过程中写到新的文件夹（固定的），洗澡的时候不知道怎么想起来，准备改成一个 random 文件夹或者带有特定字符串( prefxi/suffix )，项目关闭 通过 hook 去删除创建的文件夹，还有 lazy create 的方法可不可行。

另外就是

-  spring config refresh 对 一些 bean 的影响
- 另外我在启动时将 io 写到文件夹后，运行过程中文件夹被删除了怎么办？

