---
title: "Domain Name System"
date: 2021-12-01T00:32:43+08:00
draft: true
toc: true
images:
tags: 
  - net
---

TCP/IP 网络通过 IP 地址来确定通信对象(网卡)，IP 地址不方便人类读写，服务器名称就容易很多了。

**如果 Web 服务器使用了虚拟主机功能，有可能无法通过 IP 地址来访问。**

域名可能比 IP 地址长的多(<=255)，**对中间网络设备有一定的压力**，DNS 机制同时方便了人类和机器的读写、转换，**是一个将域名和 IP 地址相互映射的分布式缓存数据库**。

但是 **DNS 的功能并不仅限于域名和 IP 关联**，还可以关联 邮件地址和邮件服务器，以及为各种信息关联相应的名称。

## 基本工作原理

### 接受客户端的查询请求

客户端查询消息内容

- 域名

- Class 

  最早的方案，DNS 在互联网以外的其他网络中的应用也被考虑到，用来识别网络信息，现在永远代表互联网 **IN**

- 记录类型

  表示域名对应何种类型的记录

  - A             Address 域名指向 IP 地址
  - CNAME 域名指向域名，在由另外一个域名提供 IP 地址
  - MX          Mail eXchange 域名对应邮件服务器
  - NS          将子域名交给其他 DNS 服务商解析
  - AAAA     将于明指向一个 IPv6 地址
  - SRV        服务定位器，用来标识某台服务器使用了某个服务，常见于微软系统的目录管理
  - TXT        对域名进行标识和说明，绝大多数 TXT 记录是用来做 SPF 记录(反垃圾邮件) 

  上面常用记录类型参考 cloud.tencent，更多可以参考维基百科 。

### DNS 资源记录表（TTL）

| 域名             | Class | 记录类型 | 响应数据 |
| ---------------- | ----- | -------- | -------- |
| liguangchang.cn  | IN    | A        | 1.2.3.4  |
| liguangchang.cn. | IN    | MX       | 1.2.3.4  |

DNS 根据域名查询对应表中的记录进行响应（**可能是多条**）

**MX 记录类型，存在优先级，一个邮件地址对应多个邮件服务器时，需要根据游戏那集判断哪个邮件服务器是优先的，优先级数值越小越优先。**

### 层次结构

由于互联网网络设备数量的庞大，DNS 服务器通过划分层次结构来管理大量的信息。

**域名用 句号 分隔出不同层次之间的界限，越靠右的位置层级越高，默认省略的根域名 .**

今天就发现很多网站地址加上根域名就会出现 403、415、CORS、404、网页不安全的问题，暂时无解。

**分层结构就出现了上下级域，上级域管理子域的 DNS 服务器的 IP 地址，将子域的 服务器 IP 地址注册到自己的 DNS 服务器中，自底向上注册，这样顶层的根域就可以向下按照划分的域来匹配出域名对应的 IP 地址。**

另外所有的 DNS 服务器需要存储 根域的 DNS 服务器信息，这样客户端通过找到最近的一台 DNS 服务器都可以找到根域 DNS 服务器，自顶向下的匹配出目标 DNS 服务器得以找出 IP 。

分配给根域 DNS 服务器的 IP 地址在全世界仅有 13 个（一个 IP 下有多台服务器），13 个服务器几乎不发生变化。

### 请求过程（*recursive*）

如果要请求的域名是 www.liguangchang.cn

1. 客户端请求最近的 DNS 服务器
2. 最近的 DNS 服务器存储有根域 DNS 服务器，将请求转发
3. 根域 DNS 服务器此时就是多叉树的根节点，根据域名结构可以判断出第一级的域 cn，响应 cn 域的 IP 地址
4. 客户端收到响应，询问 cn 域 DNS 服务器，repeat
5. 直到 www.liguangchang.cn

### 缓存加速

上面是 DNS 工作的基本原理，真是互联网中工作方式，一台 DNS 服务器可以管理多个域的信息，并不是每一个域都有一台 DNS 服务器，**中上级域 和 下级域 可能共享一台 DNS服务器，这样就可以直接返回下一级 DNS 服务器的信息**。

DNS 服务器的缓存功能，进一步减少了多次询问，并且**查询域名不存在时，不存在响应结果也会被缓存**。

**注册信息改变，缓存信息可能不是正确的，缓存也是需要设置 TTL 的，DNS 服务器会告知客户端这一响应的结果是来自缓存中还是负责管理改域名的 DNS 服务器**。

**平时编码可以根据实际场景是否将空记录写入缓存，或者将空记录写入缓存设置较小的 TTL，根据实际情况 trade off。**

## 解析切流量

因为缓存的存在，修改域名解析记录DNS缓存服务器不可能立即生效，权威 DNS服务器可以立即生效，为了防止数据流向错误，可以提前将域名解析记录 TTL 改小。

## 本地 DNS 缓存

``cat /etc/resolv.conf``

## DNS 报文



## dig

13 根域名服务器（IP）

```
$ dig . NS

; <<>> DiG 9.10.6 <<>> . NS
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13777
;; flags: qr rd ra; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;.				IN	NS

;; ANSWER SECTION:
.			442720	IN	NS	k.root-servers.net.
.			442720	IN	NS	m.root-servers.net.
.			442720	IN	NS	c.root-servers.net.
.			442720	IN	NS	d.root-servers.net.
.			442720	IN	NS	j.root-servers.net.
.			442720	IN	NS	b.root-servers.net.
.			442720	IN	NS	l.root-servers.net.
.			442720	IN	NS	a.root-servers.net.
.			442720	IN	NS	h.root-servers.net.
.			442720	IN	NS	i.root-servers.net.
.			442720	IN	NS	e.root-servers.net.
.			442720	IN	NS	g.root-servers.net.
.			442720	IN	NS	f.root-servers.net.

;; Query time: 6 msec
;; SERVER: 192.168.18.1#53(192.168.18.1)
;; WHEN: Wed Dec 01 02:01:23 CST 2021
;; MSG SIZE  rcvd: 228
```

## DNS 劫持

## jmDNS



[腾讯云DNS解析记录类型说明]: https://cloud.tencent.com/document/product/302/38661
[维基百科DNS记录类型]: https://zh.wikipedia.org/wiki/DNS%E8%AE%B0%E5%BD%95%E7%B1%BB%E5%9E%8B%E5%88%97%E8%A1%A8

