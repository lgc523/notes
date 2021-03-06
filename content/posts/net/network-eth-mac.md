---
title: "MAC"
date: 2021-08-26T22:17:43+08:00
draft: true
toc: true
images:
tags: 
  - net
---

TCP/IP 协议中，网络访问层对应OSI 七层网络模型的物理层和数据链路层。

- ###### 物理层

  物理层处于OSI七层模型的最底层，在进行数据传输时，提供传送数据的通路和可靠的环境，对于计算机来说，物理层对应的就是网络适配器。

  根据网络适配器存在的方式，可以分为 物理网络适配器(有线网卡、无线网卡)，虚拟网络适配器(宽带拨号、VPN连接)。

  ```
  显示网络适配器细腻下
  netwox 169
  #
  Lo0 127.0.0.1 notether
  Lo0 ::1 notether
  Eth0 172.21.0.9 52:54:00:3F:B9:45
  Eth0 fe80::5054:ff:fe3f:b945 52:54:00:3F:B9:45
  Eth1 172.17.0.1 02:42:BC:9E:F3:B2
  Eth1 fe80::42:bcff:fe9e:f3b2 02:42:BC:9E:F3:B2
  Unk0 fe80::44ee:48ff:fe4c:4640 notether
  Unk1 fe80::d4e9:a3ff:fee2:aab7 notether
  Unk2 fe80::2cf4:36ff:fe09:fa6f notether
  ```

  L0 表示回环接口，是虚拟网络适配器

  Eth 是以太网络适配器

  如果同类型设备有多个，会在后面添加数字编号，从0开始，表示该类型的网络接口的第一个设备。

- 数据链路层

  数据链路层介于OSI七层网络模型物理层和网络层之间，为网络层提供数据传送服务，定义了数据传输的起始位置，并且通过一些规则来控制这些数据的传输，以保证数据传输的正确性。

  - 介质访问控制 (Media Access Control，MAC): 提供与网络适配器的接口，网络适配器驱动通常被称为MAC驱动，网卡地址称为MAC地址。
  - 逻辑链路控制 (Logical Link Control，LLC)： 这个子网对经过子网传递的桢进行错误检查，并且管理子网上通信设备之间链路。

## 网络类型

- IEEE 802.3 以太网，有线局域网
- IEEE 802.11 无线网，WIFI 网络
- IEEE 802.16 WiMAX ，移动通信长距离无线连接的技术
- 点到点协议 PPP ，使用 Modem 通过电话线进行连接的技术，eg：拨号

## MAC地址

物理地址是一种标识符，标识网络中的每个设备。由于网络设备对物理地址的处理能力有限，物理地址只在当前局域网内有效，所以接收方的物理地址都必须存在于当前局域网内，否则会导致发送失败。

### 格式

数据包中都会包含发送方和接收方的物理地址，数据包从起始地发送到目的地，为了能够正确的将数据包发送出去，必须要求MAC地址具有唯一性，因此MAC地址都是由生产厂家生产时固化在网络硬件中，是硬件的预留地址。

MAC地址采用十六进制表示，共6个字节(48位)，长度为48bit，整个地址可以分为前24位，后24位。

前24位称为组织唯一标识符（Organizationally Unique Identifier ，OUI），是由IEEE 的注册管理机构给不同厂家分配的代码，区分了不同的厂家。

后24位是厂家自己分配的，称为拓展标识符，同一个厂家生产的网卡中 MAC 地址后24位是不同的。

### 查询MAC厂商

MAC地址的前24位可以判断出硬件的生产厂商和生产地址，可以在 http://mac.51240.com

### 查看网络主机 MAC 地址信息

```
#显示网络主机 MAC 地址
netwox 5 -i ip
#显示局域网所有的主机MAC地址 [显示扫描进度]
netwox 5 -i ip/24 [-u] 
```

### 根据MAC地址获取主机信息

```
netwox 4 -e MAC地址 [--ip/--host/--title]
```

## 以太网

计算机网络的拓扑结构是引用拓扑学中研究与大小、形状无关的点、线关系的方法。把网络中的计算机和通信设备抽象为一个点，把传输介质抽象为一条线，由点和线组成的几何图形就是计算机网络的拓扑结构。

以太网主要分为总线型和星型

- 总线型 所有的计算机通过一条同轴电缆进行连接 （被淘汰）
- 星型 所有的计算机都连接到一个中央网络设备上(如交换机)

### 工作机制

以太网采用冲突检测的载波侦听多路访问(CSMA/CD)机制，以太网中所有节点都可以看到在网络中发送的所有信息，因此，以太网是一种广播网络。

需要判断计算机何时可以把数据发送到访问介质。

通过使用CSMA/CD,所有计算机都可以监视传输介质的状态，在传输之前等待线路空闲。

如果两台计算机尝试同时发送数据，就会发生冲突，计算机会停止发送，等待一个随机的时间间隔，然后再次尝试发送。

当以太网中的一台主机要传输数据时，工作过程：

1. 监听信道上是否有信号在传输，如果有，表示信道处于忙状态，继续侦听，直到信道空闲为止
2. 若没有监听到任何信号，就传输数据
3. 传输数据的时候继续监听，如果发现冲突，则执行退避算法。随机等待一段时间后，重新执行步骤(1)
4. 当冲突发生时，涉及冲突的计算机会返回监听信道状态。若未发生冲突，则表示发送成功。

## 以太桢结构

以太网链路传输的数据包称为以太桢，在以太网中，网络访问层的软件必须把数据转换成能够通过网络适配器硬件进行传输的格式。

https://textik.com/#f53d927f9a172c50

### 前同步码

用来使接受端的适配器在接受MAC桢时能够迅速调整时钟频率，使它和发送端的频率相同。

### 桢开始定界符

桢的起始字符

### 目的地址

接受桢的网络适配器的物理地址(MAC地址)，当网卡接受到一个数据桢时，首先会检查该桢的目的地址，是否与当前的网络适配器的物理地址相同，如果相同，就会进一步处理，不同直接丢弃。

### 源地址

发送桢的网络适配器的物理地址(MAC地址)

### 类型

上层协议的类型，由于上层协议众多，所以在处理数据的时候必须设置该字段，标识数据交付哪个协议处理

### 数据

也叫做有效载荷，表示交付给上层的数据。

以太网数据桢数据长度最小为46字节，最大为1500字节。如果不足46字节时，会填充到最小长度。最大值也叫最大传输单元(MTU)

在 Linux 中，ifconfig 可以查看该值，通常为 1500。【mtu 1500】

### 桢检测序列 FCS

检测该桢是否出现差错

发送方计算桢的冗余码校验(CRC)值，把这个值写到桢里。

接收方计算机重新计算CRC，与FCS字段的值进行比较，如果两个值不相同，则表示传输过程中发生了数据丢失或改变，这个时候就需要重新传输这一桢。

### 工作机制

当以太网软件从网络层接收到数据报之后，需要完成的操作

1. 根据需要把网际层的数据分解位较小的快，以符合以太网数据段的要求。以太网的整体大小必须在 64-1518 字节之间(不包含前导码)，有些系统支持更大的桢，最大可以支持9000字节。
2. 把数据块打包成桢，每一帧都包含数据及其他信息，这些信息是以太网网络适配器处理桢所需要的。
3. 把数据桢传递给对应于OSI模型物理层的底层组件，后者把桢转化为比特流，并且通过介质发送出去
4. 以太网是那个的其他网络适配器接收到这个桢，检查其中的目的地址。如果目的地址与网络适配器的地址匹配，适配器软件就会处理接收到的桢，把数据协议传递给协议栈中较高的层。

### 以太桢洪水攻击

​       交换机为了方便数据接收，通常会存储各个端口所对应的MAC地址，形成一张表。当交换机收到计算机发来的以太桢时，就会查看桢中的源MAC地址，并查找存储的表，如果表中存在该MAC地址，就直接转发数据。如果没有，则将该MAC地址存入该表中。

​        当其他计算机向这个MAC地址发送数据时，可以快速决定向哪个端口发送数据。由于该表不可能是无穷大的，所以当达到一定数量时，将不会存储其他新的MAC地址。再有新的主机发来数据桢时，部分交换机将不再查找对应的端口，而是以广播的形式转发给所有的端口，这样，其他主机就可以接收到该数据桢了。

netwox 75 用来实现以太桢洪水攻击功能，可以伪造大量的以太网数据包，填满交换器的存储表，使交换机失去正确的转发功能.

**mac装好去公司试一下看看**

## 网络配置信息

计算机的网络配置信息包含 网络设备接口，IP地址，MAC地址和掩码等信息

netwox 1 显示当前主机的网络接口信息

```
################################### Devices ###################################
nu dev   ethernet_hwtype   mtu   real_device_name
1  Lo0   loopback          16384 lo0
2  Unk0  unknown           1280  gif0
3  Unk1  unknown           1280  stf0
4  Eth0  1E:00:DA:11:3B:48 1500  anpi0
5  Eth1  1E:00:DA:11:3B:49 1500  anpi1
6  Eth2  1E:00:DA:11:3B:28 1500  en3
7  Eth3  1E:00:DA:11:3B:29 1500  en4
8  Eth4  36:DD:DB:59:C0:80 1500  en1
9  Eth5  36:DD:DB:59:C0:84 1500  en2
10 Eth6  A2:78:17:A4:3A:A7 1500  ap1
11 Eth7  A0:78:17:A4:3A:A7 1500  en0
12 Eth8  36:DD:DB:59:C0:80 1500  bridge0
13 Eth9  4E:8C:1E:DE:93:FF 1500  awdl0
14 Eth10 4E:8C:1E:DE:93:FF 1500  llw0
15 Unk2  unknown           1380  utun0
16 Unk3  unknown           2000  utun1
17 Unk4  unknown           0
```



- nu 设备编号
- dev 设备接口名称的简单形式
- ethernet_hwtype 以太网地址或硬件类型
- mtu MTU 值
- real_device_name 设备接口名称的真正形式

```
##################################### IP ######################################
nu ip             /netmask                    ppp point_to_point_with
1  127.0.0.1      /255.0.0.0                  0
11 192.168.18.239 /255.255.255.0              0
```

- nu 与此地址关联的设备编号
- ip ip 地址
- netmask 子网掩码

- ppp 点对点的地址
- point_to_point_with 远程端点的地址

```
############################## ArpCache/Neighbor #############################
nu ethernet          ip
11 52:62:D7:96:72:34 192.168.18.162
11 80:12:DF:91:32:B8 192.168.18.1
11 A0:78:17:A4:3A:A7 192.168.18.239
11 A0:78:17:A4:3A:A7 fd13:4ab0:d619:0:8c2:c732:8ef8:d78d
17 01:00:5E:00:00:FB 224.0.0.251
17 01:00:5E:7F:FF:FA 239.255.255.250
```

- nu 与此条目关联的设备编号
- ethernet 计算机的以太网地址
- ip 计算机的ip 地址

```
#################################### Routes ###################################
nu destination    /netmask         source              gateway           metric
1  127.0.0.1      /255.255.255.255 local                                      0
11 8.8.8.8        /255.255.255.255 192.168.18.239      192.168.18.1           0
11 17.57.145.136  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 17.57.145.170  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 23.205.97.4    /255.255.255.255 192.168.18.239      192.168.18.1           0
11 35.190.60.146  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 35.227.202.26  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 35.244.159.8   /255.255.255.255 192.168.18.239      192.168.18.1           0
11 36.110.238.56  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 58.222.35.204  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 61.151.165.101 /255.255.255.255 192.168.18.239      192.168.18.1           0
11 67.195.228.56  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 69.173.158.64  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 101.89.15.106  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 103.231.98.196 /255.255.255.255 192.168.18.239      192.168.18.1           0
11 103.254.191.161/255.255.255.255 192.168.18.239      192.168.18.1           0
11 104.18.2.36    /255.255.255.255 192.168.18.239      192.168.18.1           0
11 106.117.249.62 /255.255.255.255 192.168.18.239      192.168.18.1           0
11 113.24.194.97  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 115.171.85.13  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 115.171.155.241/255.255.255.255 192.168.18.239      192.168.18.1           0
11 116.211.227.21 /255.255.255.255 192.168.18.239      192.168.18.1           0
11 120.240.48.239 /255.255.255.255 192.168.18.239      192.168.18.1           0
11 123.151.79.29  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 124.126.3.87   /255.255.255.255 192.168.18.239      192.168.18.1           0
11 124.126.179.253/255.255.255.255 192.168.18.239      192.168.18.1           0
11 220.181.3.162  /255.255.255.255 192.168.18.239      192.168.18.1           0
11 192.168.18.239 /255.255.255.255 local                                      0
11 192.168.18.0   /255.255.255.0   192.168.18.239                             0
1  127.0.0.0      /255.0.0.0       127.0.0.1                                  0
11 0.0.0.0        /0.0.0.0         192.168.18.239      192.168.18.1           0
11 fd13:4ab0:d619::/0              192.168.18.239      ::                     0
```

- nu 与此条目关联的设备编号
- destination 目标地址
- netmask 掩码
- source 源IP地址或本地路由
- gateway 网关
- metric 路线度量
