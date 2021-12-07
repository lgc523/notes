---
title: "RTMP HLS"
date: 2021-12-07T02:01:23+08:00
draft: true
toc: true
images:
tags: 
  - nginx
---

## nginx -rtmp-module 

- 支持 RTMP、HLS、MPEG-DASH 直播
- 支持 RTMP、HLS 点播
- 可以将一次直播分为多个视频文件存储
- 支持 H.264 视频编解码或 AAC 视频编解码
- 支持 FFmpeg 命令内嵌
- 支持回调 HTTP
- 可以使用 HTTP 对直播进行控制，删除/录播。
- 具有更优秀的缓存技术，确保在效率与解码之间达到平衡，获得更好的效果。
- 支持更多的操作系统

## Real Time Message Protocol 

实时消息传送协议，是 Adobe 公司为 Flash 播放器和服务器之间传输音/视频和数据而开发的私有协议。

RTMP 是一个开放的规范，可以通过 OpenAMF，SWF，FLV 和 F4V 格式，提供与 Adobe Flash Player 兼容 的视音频和数据。

### feats

- RTMP 是应用层协议，需要依靠底层可靠的传输层协议来保证信息传输的可靠性。

- 建立完基于传输层协议的链接后，需要客户端和服务端 <握手>来建立基于传输层之上的 RTMP 连接。

  连接会传输一些控制信息，eg SetChunkSize 、SetACKWindowSize

  CreateStream 命令会创建一个 Stream 链接，用于传输具体的音/视频等数据，以及控制信息传输的命令信息。

- RTMP 协议在传输时会对数据进行格式化，被格式化的消息被称为 RTMP Message。

- 为了更好的实现多路复用、分包和信息的公平性，发送端会把 Message 划分为带有 Message ID 的 Chunk 。每个 Chunk 可能是一个单独的 Message，也可能是 Message 的一部分，在接收端会根据 Chunk 中包含的 data 的长度、message id 和 message 的长度，把 chunk 还原成完整的 Message，从而实现信息的收发。

### handshake

RTMP 协议的 “握手” 由**三个固定大小**的 chunk 组成。

客户端和服务端个字发送三个相同的块，客户端发送的 [C0,C1,C2]，服务端发送的[S0,S1,S2]。

- 客户端要等收到 S1 之后才能发送 C2
- 客户端要等到 S2 之后才能发送其他信息 (控制信息和真实音/视频等数据）
- 服务端要等收到 C0 之后才能发送 S1
- 服务端必须要等收到 C1 之后才能发送 S2
- 服务端必须要等收到 C2 之后才能发送其他信息(控制信息和真实音/视频等数据)。



## HTTP Live Streaming

HLS 协议基于HTTP，最初是 Apple 公司针对 iPhone，iPod，iTouch 和 iPad 等移动设备开发的流，继承了很多 HTTP 的优点。

