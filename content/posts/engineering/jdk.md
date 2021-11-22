---
title: "jdk"
date: 2021-11-18T23:06:31+08:00
draft: true
toc: true
images:
tags: 
  - engineering
---

先安装多版本java，忘了为啥要安装多版本，最近看了一下 java 实现 CLI ，平时用 go 写很方便，想起来了，用java 编译跨平台低版本不支持，这次安装一下，并且记录一下版本 feat，正好极客时间出了一门课讲这个，但是我觉得没必要，直接看官网应该就行了。之前看公众号碎片化只知道 字符串块，jshell，jpackage，更大的堆，更高效的垃圾回收，正好和最近看的书比较契合。

安装版本 openjdk adopt，正好看一下 java 、虚拟机相关的历史和 Graalvm。

install

``brew install jenv``  shell 写的工具 git ``https://github.com/jenv/jenv``

``alias jv="/usr/libexec/java_home -V"``

```
Matching Java Virtual Machines (4):
    17.0.1 (arm64) "Eclipse Temurin" - "Eclipse Temurin 17" /Library/Java/JavaVirtualMachines/temurin-17.jdk/Contents/Home
    11.0.13 (x86_64) "Eclipse Temurin" - "Eclipse Temurin 11" /Library/Java/JavaVirtualMachines/temurin-11.jdk/Contents/Home
    1.8.201.09 (x86_64) "Oracle Corporation" - "Java" /Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home
    1.8.0_201 (x86_64) "Oracle Corporation" - "Java SE 8" /Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home
/Library/Java/JavaVirtualMachines/temurin-17.jdk/Contents/Home
```

```
To activate jenv, add the following to your ~/.zshrc:
  export PATH="$HOME/.jenv/bin:$PATH"
  eval "$(jenv init -)"
```

参数配置看提示就行了

