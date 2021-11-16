---
title: "Netwox"
date: 2021-08-25T23:33:42+08:00
draft: true
toc: true
images:
tags: 
  - net
---

## install

  - Install libpcap yourself (download it from http://www.tcpdump.org/
    or use a package for your system)
  - Install libnet yourself (download it from
    http://www.packetfactory.net/libnet or use a package for your system)
  - Install Tcl/Tk yourself (download it from http://www.tcl.tk/ or use
    a package for your system)
  - Run netwib/netwox/netwag automatic installer :
      ./installunix.sh  [it asks some questions]



```
 #libpcap
 yum list |grep pcap
 yum install libpcap.x86_64 
 #libnet
 yum list |grep  net
 yum install libdnet.x86_64
 ldconfig -p |grep net
 #Tcl/Tk
 yum install tcl.x86_64 
 yum install  tk.x86_64
 yum install xterm.x86_64
 #netwox
 wget -c https://sourceforge.net/projects/ntwox/files/netwib%20netwox%20and%20netwag/5.39/netw-ib-ox-ag-5.39.0.tgz
 tar -xvf netw-ib-ox-ag-5.39.0.tgz
 cd netw-ib-ox-ag-5.39.0
 bash installunix.sh
```

## menu

 0 - leave netwox
 3 - search tools
 4 - display help of one tool
 5 - run a tool selecting parameters on command line
 6 - run a tool selecting parameters from keyboard
 a + information
 b + network protocol
 c + application protocol
 d + sniff (capture network packets)
 e + spoof (create and send packets)
 f + record (file containing captured packets)
 g + client
 h + server
 i + ping (check if a computer if reachable)
 j + traceroute (obtain list of gateways)
 k + scan (computer and port discovery)
 l + network audit
 m + brute force (check if passwords are weak)
 n + remote administration
 o + tools not related to network

费劲

```
0：退出 netwox 工具。
3：搜索工具，用来搜索与指定信息相关的模块。
4：显示指定模块的帮助信息。
5：在命令行中输入指定模块的参数选项并运行。
6：从键盘输入指定模块的参数选项并运行。
a：显示信息。
b：显示网络协议下相关的模块。
c：显示应用程序协议下相关的模块。
d：显示与嗅探数据包相关的模块。
e：显示与创建和发送数据包相关的模块。
f：显示与进行数据包记录相关的模块。
g：显示与客户端相关的模块。
h：显示与服务器相关的模块。
i：显示与检测主机连通性相关的模块。
j：显示与路由跟踪相关的模块。
k：显示与扫描计算机和端口相关的模块。
l：显示与审计相关的模块。
m：显示与暴力破解相关的模块。
n：显示与远程管理相关的模块。
o：显示其他模块。
```

熟悉之后可以直接 netwox 模块编号 options

eg: netwox 1 -i 查看ip
