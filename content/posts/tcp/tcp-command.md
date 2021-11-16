---
title: "TCP Command"
date: 2021-09-23T01:25:43+08:00
draft: true
toc: true
images:
tags: 
  - tcp
---

## ip

```
curl https://api.myip.com | json
```

## nc

我喜欢用这个

```
nc -vz -w 1 ip port
-w 1  # TTL 1s 
```

## ss

ss  

```
ss -ltp  | grep  sshd # 查看进程的端口占用
ss -p    | grep  22   # 查看端口被哪个进程占用
ss -tenp | grep  22   # 列出某个端口上的tcp连接
ss -t  #所有tcp连接
ss -tl #所有处于监听状态的TCP连接
ss -u  #所有udp连接
ss -p  #列出连接时显示进程名字和pid
ss -s  #统计socket

```

## netstat

```
netstat -tunpal | grep sshd | grep LISTEN # 找出进程监听的端口号
netstat -tunpal | grep 22   | grep LISTEN # 找出某个端口号被哪个进程占用
```

## lsof

这个命令可以表明，socket 其实是五元组(协议)，同一个端口号可以被多个进程的不同协议绑定

```
lsof -p pid     #列出进程打开的文件
lsof -i tcp     #列出所有tcp连接
lsof -i udp
lsof -i :55555  #列出占用端口的连接
lsof -i tcp:55555
lsof -i udp:55555
lsof -c name    #列出进程名开头以 name 打开的文件
```



## sock



## tcpdump

