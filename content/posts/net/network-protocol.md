---
title: "网络协议"
date: 2021-08-25T23:12:02+08:00
draft: true
toc: true
images:
tags: 
  - net
---

网络协议：在网络中的不同设备之间进行数据交换建立的规则、标准或者约定的集合，规定了通信时必须采用的格式和这些格式代表的意义。接收方和发送方采用的协议必须一致，否则一方将无法识别另一方的信息。

## OSI协议分层

OSI协议是基于1984年国际标准化组织(ISO)发布的 ISO/IEC 7498 标准定义的网络互联的七层框架。

- 应用层

  为应用程序提供服务并规定应用程序中相关的通信细节，HTTP，SMTP，Telnet

- 表示层

  将应用处理的信息转换为适合网络传输的格式，或将下一层的数据转换为上层能够处理的格式。

  主要负责数据格式的转换，确保一个系统的应用层信息可被另一个系统应用层读取。

- 会话层

  负责建立和断开通信连接(数据流动的逻辑通路)，以及记忆数据的分隔等数据传输相关的管理。

- 传输层

  只在通信双方的节点上(设备终端)进行处理，无需在路由器上处理

- 网络层

  将数据传输到目标地址，主要负责寻址和路由选择，还可以实现拥塞控制、网际互联等功能。

- 数据链路层

  负责物理层面上互连的节点间通信传输，一个以太网相连的两个节点之间的通信。

  作用包括物理地址寻址、数据的成桢、流量控制、数据的检错和重发等

- 物理层

  利用传输介质为数据链路层提供物理连接，实现比特流的透明传输。

## TCP/IP协议层次

- 应用层

  xxx

- 传输层

  为两台主机上的应用程序提供端到端的通信，提供流量控制、错误控制、确认服务。

- 网际层

  提供独立于硬件的逻辑寻址，让数据能够在具有不同的物理结构的子网之间传递。

  负责寻址和路由选择，还可以实现拥塞控制、网际互联等功能。

- 网络访问层

  提供了物理网络连接的接口。

  针对传输介质设置数据格式，根据硬件的物理地址实现数据的寻址，对数据在物理网络中的传递提供错误控制。



## 网络访问层

Todo