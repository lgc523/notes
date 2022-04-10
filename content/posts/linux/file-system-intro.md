---
title: "Linux 文件系统"
date: 2022-03-02T00:30:43+08:00
draft: true
toc: true
tags: 
  - linux
  - file system
---

## 查看文件系统

``ls -l /lib/modules/$(uname -r)/kernel/fs`` 查看支持的文件类型

``cat /proc/filesystems 已载入内存的磁盘类型``

``parted -l``分区表信息

``blkid``  查看块设备信息

``df -T ``  查看所有磁盘的文件系统类型

``df -T -h``  查看磁盘占用、类型、挂载点，``df --type=devtmpfs`` 筛选

``fdisk -l`` 查看所有被识别的磁盘

``lsblk`` 查看块设备，-d 只列出硬盘，不列出分区

``lshw``  查看硬盘的详细信息

``lsscsi`` 查看 SCSI 硬盘信息



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

## /lost+found

使用标准的 ext2、ext3、ext4、文件系统格式会产生的一个目录，目的在于当文件系统发生错误时，将一些遗失的片段放置到这个目录下，xfs 文件系统不会存在。

## Ext4 布局

ext4 文件系统将磁盘空间划分为若干个子空间（块组），每个子空间划分为等份的逻辑块，逻辑块是最小的管理单元，逻辑块的大小由用户在格式化时确定。





![ext4-layout](https://s2.loli.net/2022/03/02/XUMzfhbiuLJ5SxN.png)



## 常见文件系统

### 本地文件系统

基于磁盘的普通本地文件系统

- Ext4
- XFS
- ZFS
- Btrfs

Btrfs 和 ZFS 不仅可以管理一块磁盘，还可以实现多块磁盘的管理，另外实现了冗余管理，可以避免磁盘故障导致的数据丢失。

### 虚拟文件系统/伪文件系统

伪文件系统是 linux 中的概念，虚拟文件系统，不会持久化数据，是内存中的文件系统。是以文件系统的形态实现用户与内核数据交互的接口，常见的伪文件系统由 proc、sysfs、configs...

linux 中 伪文件主要实现内核与用户态的交互，iostat 本质是通过访问 /proc/diskstats 文件获取信息，其内容是内核中对磁盘访问的统计，是内核某些数据结构的实例。

## linux 文件种类

ls 查看文件权限，第一个字母

- device
    - b， block 区块设备文件，存储数据、提供系统随机存取的接口设备
    - c ，character 字符设备文件，串行端口的接口设备，eg 键盘、鼠标
- sockets
    - s 数据接口文件，通常用在网络上数据交换，/run,/tmp
- fifo，pipe
    - p 数据输送文件，主要解决多个程序同时读写一个文件造成的错误问题

## 文件属性与权限

write 文件权限不含删除该文件。

常见的 rwx 【 u:user g:group o:other】，可以通过 +、-、= 进行修改，chmod u=rwx,go=rx xxx

chgrp（change group）、chown （change owner） 修改用户组、拥有者，chown -R（recursive） 修改所有子目录。

一般可以用 chown user.group file 修改文件的拥有者和用户组，为了避免用户名存在 . ，使用 user:group 更好，chown .group file 也可以。

**lsattr，chatter 命令可以控制文件修改、删除的权限**

## 目录权限

- r  read contents of directory 读取目录列表权限
- w modify contents of directory
  - 建立新的文件与目录
  - 删除已经存在的文件与目录
  - 重命名、移动位置
- x access directory
  - 能够进入目成为工作目录

## 特殊权限

chmod 修改 **suid、sgid、sbit** 权限

## last

用户登陆数据存放在 /var/log/wtmp，使用 last 可以读。

## 文件名限制

单一文件名或目录的最大容许文件名为 255 字节。(ext2、ext3、ext4、xfs)

## type hash

之前 只知道 type comman  可以看 shell 命令的类型信息(buildin、alias、keyword)，今天发现输出 command is hashed 

gg 发现 shell 会把执行过的命令用哈希表及路况，下次执行就直接查看哈希表，找到就用命令的全路径，没找到再去 $PATH 中查找，同时会添加到哈希表中。

``hash -r `` 清空哈希表

## FHS

``FileSystem Hierarchy Standard``，规定每个特定的目录下放置什么数据。

### 应该放置的文件内容

- /bin 存放执行文件，命令可以被 root 与一般账户使用，eg: cp

- /boot 启动使用到的文件

- /dev 设备与接口设备

- /etc 配置文件

  - 一般 root 有权修改，一般用户可见
  - /etc/opt  /opt 第三方辅助软件的配置文件
  - /etc/xml XML 格式相关的配置文件

- /lib 系统函数 

  - 启动时以及 /bin、/sbin 命令会滴哦啊用的函数库

- /media 可删除设备 软盘、光盘等设备

- /mnt 暂时挂载额外的设备

- **/opt 辅助软件目录 或者 /usr/local**

- run 早期 FHS 规定启动后所产生的各项信息要放置到 /var/run 目录下，新版规范到 /run 下面，/run 下面可以使用内存来模拟，性能更好?

  ![image-20220408000321924](https://s2.loli.net/2022/04/08/HGT2yvtO9NpZ4w5.png)

​		/var/run  链接引用了 /run 目录，**tempfs 文件系统是存储在内存中的临时文件系统，机器关闭就被清空**。

​		/var 通常存放 daemon 进程的 pid，eg: sshd，crond

- /sbin
  - 启动过程中的命令 eg: ifconfig
- /srv
  - Service 网络服务启动后需要使用的数据目录，一般需要提供到网络浏览的，可以将 hugo public 放到 /srv/html 下面
- /tmp
  - 一般用户或者正在执行的程序暂时放置文件的地方，FHS 建议在启动时删除 /tmp 下的数据

### 建议可以存在的目录

- /home 系统默认用户根目录
- lib<qual> lib 格式不同的二进制函数库，/lib64
- /root 

### 应放置的内容

- /lost+found
  - ext2，ext3，ext4 文件系统格式存放文件系统错误时一些遗失的片段
  - 只有 root 有权限
- /proc
  - 虚拟文件系统，放置的数据都在内存当中，eg: 内核、进程信息、外接设备、网关的状态
  - /proc/cpuinfo
  - /proc/dma
  - /proc/interrupts
  - /proc/ioports
  - /porc/net/
- /sys
  - 同 /proc ，虚拟文件系统，主要记录内核与系统硬件信息相关的内容，包括目前已加载的内核模块与内核检测到的硬件设备信息。

许多 Linux 发行版已经将许多非必要的文件移出 /usr 之外，**/bin、/lib、/lib64、/sbin 都链接到了 /usr。**

### /usr

addr of UNIX Software Resource ,是系统的软件资源.

- /usr/bin
  - 所有一般用户可以使用的命令，改目录下不应该有子目录
- /usr/lib
  - soft symbolic /lib
- /usr/local
  - 辅助软件
- /usr/sbin
  - soft symbolic /sbin ，常见网络服务器软件的命令
- /usr/share
  - 只读的文数据文件，数据文件几乎不区分硬件架构
  - /usr/share/man
  - /usr/share/doc
  - /usr/share/zoneinfo

### 建议可以存在的目录

- /usr/include
  - c/c艹 等程序的头文件
- /usr/libexec
  - 某些不被一般用户常用的执行文件或脚本
- /usr/lib<qual> 
  - Soft symbolic /lib
- /usr/src
  - 源代码，eg: kernels

### /var

针对目录架构定义的三层目录标准

- / root 目录，与启动系统有关
- /usr  unix software resource，与软件安装/执行有关
- /var variable [与系统运行过程有关]()
