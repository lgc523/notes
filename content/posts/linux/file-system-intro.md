---
title: "File System Intro"
date: 2022-03-02T00:30:43+08:00
draft: true
toc: true
tags: 
  - linux
  - file system
---

查看 linux 支持的文件系统

``ls -l /lib/modules/$(uname -r)/kernel/fs``

``cat /proc/filesystems 已载入内存``



## ext

linux extended file system,extfs

Ext3/Ext4 是 Ext2 的升级版，减少一致性检查，为了快速恢复文件系统，增加了日志功能，ext2 称为索引式文件系统，Ext3/Ext4 称为日志式文件系统。

## 文件系统层次

底层硬件设备 -> 中间层是存储设备驱动器和操作系统 -> 顶层是文件系统

中间层将底层存储抽象为逻辑连续的线性空间

顶层的文件系统暴露给用户一个友好的层级结构，对线性空间进行管理和抽象。

**对于文件系统，所有的文件都是字节流，不关注文件的格式和内容。**

操作系统层面建立了文件格式与软件的关联使得用户可以打开文件，关联关系被破坏或者不存在相应的软件都无法打开文件。

？关联关系在哪里？

目录存储了文件名和 inodeId 的映射关系，**硬连接和原文件**的 inodeId 一样。

## 解决的问题

- 解决多进程访问磁盘冲突
- 管理磁盘空间

## 构建在块设备上

文件系统还可以构建在任何块设备上(disk、分区、LVM 卷、RAID...)、网络上。

在文件上构建块文件系统

```
dd if=/dev/zero of=./img.bin bs=1M count=1
格式化，构建 ext2 文件系统
mkfs.ext2 img.bin
dumpe2fs 查看文件系统的描述信息
或者
借助 Linux 的循环设备，将文件虚拟成块设备，将块设备挂载到目录树中
losetup /dev/loop10 ./img.bin
mkdir /tmp/ext2
mount /dev/loop10 /tmp/ext2
向该目录中拷贝文件，会看到 inodeId 连续递增
```

文件系统实现了对线性存储空间的管理，存储空间既可以是磁盘等快设备，还可以是一个文件。

## Ext4 布局

## 常见文件系统
