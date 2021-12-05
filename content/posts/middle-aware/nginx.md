---
title: "Nginx"
date: 2021-08-03T22:26:02+08:00
draft: true
author: "spider"
toc: true
tags:
  - middle-aware
---
nginx 是一个高性能的 HTTP 和**反向**代理服务器，同时也是一个 IMAP/POP3/SMTP 代理服务器，可以实现负载均衡、作为邮件代理服务器来收发邮件(最早开发的目的之一)，1.9.0 以后的版本还可以作为通用的 TCP/UDP 代理服务器，也可以提供一定的缓存服务功能。

## why nginx

- 高并发、性能好、占用内存少（10000个非活动HTTP保持连接，占用2.5m），稳定
- 功能强大，提供大量功能模块，特性多
- 拓展性好，多个不同层次、不同功能、不同类型低耦合的模块组成
- 跨平台、支持内置服务器检测、网络依赖性低

## version

mainline 

## yum install

```javascript
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
sudo yum install -y nginx
systemctl enable --now nginx
systemctl status nginx
#目前笔记部署静态跨域处理
add_header Access-Control-Allow-Origin *;
add_header Access-Control-Allow-Headers X-Requested-With;
add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
```

```
网站文件存放默认目录：/usr/share/nginx/html
网站默认站点配置：/etc/nginx/conf.d/default.conf
自定义Nginx站点配置文件存放目录：/etc/nginx/conf.d/
Nginx全局配置：/etc/nginx/nginx.conf
Nginx指定配置文件启动：nginx -c nginx.conf
PID目录：/var/run/nginx.pid
错误日志：/var/log/nginx/error.log
访问日志：/var/log/nginx/access.log
```

## compile install

```
yum install gcc     编译
yum install gcc-c++ [c++模块开发]
yum install -y pcre pcre-devel [Perl Compatible Regular Expressions，Perl兼容正则表达式]
yum install -y zlib zlib-devel [对HTTP包的内容做gzip格式的压缩]
yum install -y openssl openssl-devel [sl] 

```

### configure

shell 脚本，可配置的分数可分为五大类

- 事件模块

  ```
    --with-select_module               enable select module
    --without-select_module            disable select module
    --with-poll_module                 enable poll module
    --without-poll_module              disable poll module
  ```

  

- 默认编译进入 nginx 的 HTTP 模块

  ```
    --without-http_charset_module      disable ngx_http_charset_module
    --without-http_gzip_module         disable ngx_http_gzip_module
    --without-http_ssi_module          disable ngx_http_ssi_module
    --without-http_userid_module       disable ngx_http_userid_module
    --without-http_access_module       disable ngx_http_access_module
    --without-http_auth_basic_module   disable ngx_http_auth_basic_module
    --without-http_mirror_module       disable ngx_http_mirror_module
    --without-http_autoindex_module    disable ngx_http_autoindex_module
    --without-http_geo_module          disable ngx_http_geo_module
    --without-http_map_module          disable ngx_http_map_module
    --without-http_split_clients_module disable ngx_http_split_clients_module
    --without-http_referer_module      disable ngx_http_referer_module
    --without-http_rewrite_module      disable ngx_http_rewrite_module
    --without-http_proxy_module        disable ngx_http_proxy_module
    --without-http_fastcgi_module      disable ngx_http_fastcgi_module
    --without-http_uwsgi_module        disable ngx_http_uwsgi_module
    --without-http_scgi_module         disable ngx_http_scgi_module
    --without-http_grpc_module         disable ngx_http_grpc_module
    --without-http_memcached_module    disable ngx_http_memcached_module
    --without-http_limit_conn_module   disable ngx_http_limit_conn_module
    --without-http_limit_req_module    disable ngx_http_limit_req_module
    --without-http_empty_gif_module    disable ngx_http_empty_gif_module
    --without-http_browser_module      disable ngx_http_browser_module
    --without-http_upstream_hash_module
                                       disable ngx_http_upstream_hash_module
    --without-http_upstream_ip_hash_module
                                       disable ngx_http_upstream_ip_hash_module
    --without-http_upstream_least_conn_module
                                       disable ngx_http_upstream_least_conn_module
    --without-http_upstream_random_module
                                       disable ngx_http_upstream_random_module
    --without-http_upstream_keepalive_module
                                       disable ngx_http_upstream_keepalive_module
    --without-http_upstream_zone_module
                                       disable ngx_http_upstream_zone_module                             
  ```

  

- 默认不会编译进入 nginx 的 HTTP 模块

  ```
    --with-http_ssl_module             enable ngx_http_ssl_module
    --with-http_v2_module              enable ngx_http_v2_module
    --with-http_realip_module          enable ngx_http_realip_module
    --with-http_addition_module        enable ngx_http_addition_module
    --with-http_xslt_module            enable ngx_http_xslt_module
    --with-http_xslt_module=dynamic    enable dynamic ngx_http_xslt_module
    --with-http_image_filter_module    enable ngx_http_image_filter_module
    --with-http_image_filter_module=dynamic
                                       enable dynamic ngx_http_image_filter_module
    --with-http_geoip_module           enable ngx_http_geoip_module
    --with-http_geoip_module=dynamic   enable dynamic ngx_http_geoip_module
    --with-http_sub_module             enable ngx_http_sub_module
    --with-http_dav_module             enable ngx_http_dav_module
    --with-http_flv_module             enable ngx_http_flv_module
    --with-http_mp4_module             enable ngx_http_mp4_module
    --with-http_gunzip_module          enable ngx_http_gunzip_module
    --with-http_gzip_static_module     enable ngx_http_gzip_static_module
    --with-http_auth_request_module    enable ngx_http_auth_request_module
    --with-http_random_index_module    enable ngx_http_random_index_module
    --with-http_secure_link_module     enable ngx_http_secure_link_module
    --with-http_degradation_module     enable ngx_http_degradation_module
    --with-http_slice_module           enable ngx_http_slice_module
    --with-http_stub_status_module     enable ngx_http_stub_status_module
    
    --with-http_perl_module            enable ngx_http_perl_module
    --with-http_perl_module=dynamic    enable dynamic ngx_http_perl_module
    --with-perl_modules_path=PATH      set Perl modules path
    --with-perl=PATH                   set perl binary pathname
  ```

  

- 邮件代理服务器相关的 mail 模块

  ```
    --with-mail                        enable POP3/IMAP4/SMTP proxy module
    --with-mail=dynamic                enable dynamic POP3/IMAP4/SMTP proxy module
    --with-mail_ssl_module             enable ngx_mail_ssl_module
    --without-mail_pop3_module         disable ngx_mail_pop3_module
    --without-mail_imap_module         disable ngx_mail_imap_module
    --without-mail_smtp_module         disable ngx_mail_smtp_module
  ```

  

- 其他模块

  ```
    --without-http                     disable HTTP server
    --without-http-cache               disable HTTP cache      
    --with-debug                       enable debug logging
  ```

### objs

```
|-- autoconf.err 保存 configure 执行过程中产生的结果
|-- Makefile 用于编译 nginx 工程、加入 install 后安装 nginx
|-- nginx
|-- nginx.8
|-- ngx_auto_config.h  保存一些宏
|-- ngx_auto_headers.h 保存一些宏
|-- ngx_modules.c    定义 ngx_modules 数组，指明了每个模块在 nginx 中的优先级
|-- ngx_modules.o
`-- src  存放编译产生的目标文件
    |-- core
    |   |-- nginx.o
    |   |-- ngx_array.o
    |   |-- ngx_buf.o
    |   |-- ngx_conf_file.o
    |   |-- ngx_connection.o
    |   |-- ngx_cpuinfo.o
    |   |-- ngx_crc32.o
    |   |-- ngx_crypt.o
    |   |-- ngx_cycle.o
    |   |-- ngx_file.o
    |   |-- ngx_hash.o
    |   |-- ngx_inet.o
    |   |-- ngx_list.o
    |   |-- ngx_log.o
    |   |-- ngx_md5.o
    |   |-- ngx_module.o
    |   |-- ngx_murmurhash.o
    |   |-- ngx_open_file_cache.o
    |   |-- ngx_output_chain.o
    |   |-- ngx_palloc.o
    |   |-- ngx_parse.o
    |   |-- ngx_parse_time.o
    |   |-- ngx_proxy_protocol.o
    |   |-- ngx_queue.o
    |   |-- ngx_radix_tree.o
    |   |-- ngx_rbtree.o
    |   |-- ngx_regex.o
    |   |-- ngx_resolver.o
    |   |-- ngx_rwlock.o
    |   |-- ngx_sha1.o
    |   |-- ngx_shmtx.o
    |   |-- ngx_slab.o
    |   |-- ngx_spinlock.o
    |   |-- ngx_string.o
    |   |-- ngx_syslog.o
    |   `-- ngx_times.o
    |-- event
    |   |-- modules
    |   |   `-- ngx_epoll_module.o
    |   |-- ngx_event_accept.o
    |   |-- ngx_event_connect.o
    |   |-- ngx_event.o
    |   |-- ngx_event_openssl.o
    |   |-- ngx_event_openssl_stapling.o
    |   |-- ngx_event_pipe.o
    |   |-- ngx_event_posted.o
    |   |-- ngx_event_timer.o
    |   `-- ngx_event_udp.o
    |-- http
    |   |-- modules
    |   |   |-- ngx_http_access_module.o
    |   |   |-- ngx_http_auth_basic_module.o
    |   |   |-- ngx_http_autoindex_module.o
    |   |   |-- ngx_http_browser_module.o
    |   |   |-- ngx_http_charset_filter_module.o
    |   |   |-- ngx_http_chunked_filter_module.o
    |   |   |-- ngx_http_empty_gif_module.o
    |   |   |-- ngx_http_fastcgi_module.o
    |   |   |-- ngx_http_geo_module.o
    |   |   |-- ngx_http_gzip_filter_module.o
    |   |   |-- ngx_http_headers_filter_module.o
    |   |   |-- ngx_http_index_module.o
    |   |   |-- ngx_http_limit_conn_module.o
    |   |   |-- ngx_http_limit_req_module.o
    |   |   |-- ngx_http_log_module.o
    |   |   |-- ngx_http_map_module.o
    |   |   |-- ngx_http_memcached_module.o
    |   |   |-- ngx_http_mirror_module.o
    |   |   |-- ngx_http_not_modified_filter_module.o
    |   |   |-- ngx_http_proxy_module.o
    |   |   |-- ngx_http_range_filter_module.o
    |   |   |-- ngx_http_referer_module.o
    |   |   |-- ngx_http_rewrite_module.o
    |   |   |-- ngx_http_scgi_module.o
    |   |   |-- ngx_http_split_clients_module.o
    |   |   |-- ngx_http_ssi_filter_module.o
    |   |   |-- ngx_http_ssl_module.o
    |   |   |-- ngx_http_static_module.o
    |   |   |-- ngx_http_try_files_module.o
    |   |   |-- ngx_http_upstream_hash_module.o
    |   |   |-- ngx_http_upstream_ip_hash_module.o
    |   |   |-- ngx_http_upstream_keepalive_module.o
    |   |   |-- ngx_http_upstream_least_conn_module.o
    |   |   |-- ngx_http_upstream_random_module.o
    |   |   |-- ngx_http_upstream_zone_module.o
    |   |   |-- ngx_http_userid_filter_module.o
    |   |   |-- ngx_http_uwsgi_module.o
    |   |   `-- perl
    |   |-- ngx_http_copy_filter_module.o
    |   |-- ngx_http_core_module.o
    |   |-- ngx_http_file_cache.o
    |   |-- ngx_http_header_filter_module.o
    |   |-- ngx_http.o
    |   |-- ngx_http_parse.o
    |   |-- ngx_http_postpone_filter_module.o
    |   |-- ngx_http_request_body.o
    |   |-- ngx_http_request.o
    |   |-- ngx_http_script.o
    |   |-- ngx_http_special_response.o
    |   |-- ngx_http_upstream.o
    |   |-- ngx_http_upstream_round_robin.o
    |   |-- ngx_http_variables.o
    |   |-- ngx_http_write_filter_module.o
    |   `-- v2
    |-- mail
    |-- misc
    |-- os
    |   |-- unix
    |   |   |-- ngx_alloc.o
    |   |   |-- ngx_channel.o
    |   |   |-- ngx_daemon.o
    |   |   |-- ngx_dlopen.o
    |   |   |-- ngx_errno.o
    |   |   |-- ngx_files.o
    |   |   |-- ngx_linux_init.o
    |   |   |-- ngx_linux_sendfile_chain.o
    |   |   |-- ngx_posix_init.o
    |   |   |-- ngx_process_cycle.o
    |   |   |-- ngx_process.o
    |   |   |-- ngx_readv_chain.o
    |   |   |-- ngx_recv.o
    |   |   |-- ngx_send.o
    |   |   |-- ngx_setaffinity.o
    |   |   |-- ngx_setproctitle.o
    |   |   |-- ngx_shmem.o
    |   |   |-- ngx_socket.o
    |   |   |-- ngx_time.o
    |   |   |-- ngx_udp_recv.o
    |   |   |-- ngx_udp_sendmsg_chain.o
    |   |   |-- ngx_udp_send.o
    |   |   |-- ngx_user.o
    |   |   `-- ngx_writev_chain.o
    |   `-- win32
    `-- stream
```



```
./configure --help

./configure \
--prefix=/opt/dev/nginx \
--with-http_ssl_module
```

主要生成 Makfile (objs)，输出安装信息

```
Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + using system zlib library

  nginx path prefix: "/opt/dev/nginx"
  nginx binary file: "/opt/dev/nginx/sbin/nginx"
  nginx modules path: "/opt/dev/nginx/modules"
  nginx configuration prefix: "/opt/dev/nginx/conf"
  nginx configuration file: "/opt/dev/nginx/conf/nginx.conf"
  nginx pid file: "/opt/dev/nginx/logs/nginx.pid"
  nginx error log file: "/opt/dev/nginx/logs/error.log"
  nginx http access log file: "/opt/dev/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

### make & install

```
make & make install
make 根据 configure 生成的 Mailfile 文件编译 Nginx 工程，生成目标文件、最终的二进制文件。
make install 根据 configure 执行时的参数将 nginx 部署到指定的安装目录，包括相关目录的建立和二进制文件、配置文件的复制。
```

### Check

**不启动测试配置文件是否有错误**，-q 不输出 error 级别一下的信息

```
/sbin/nginx -t
nginx: the configuration file /opt/dev/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /opt/dev/nginx/conf/nginx.conf test is successful
```

查看版本 -v

```
sbin/nginx -v
nginx version: nginx/1.20.2
```

查看编译阶段参数 -V

```
sbin/nginx -V

nginx version: nginx/1.20.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/opt/dev/nginx --with-http_ssl_module
```



### /etc/sysctl.conf

```
fs.file-max = 999999
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.ip_local_port_range = 1024 61000
net.ipv4.tcp_rmem = 4096 32768 262142
net.ipv4.tcp_wmem = 4096 32768 262142
net.core.netdev_max_backlog = 8096
net.core.rmem_default = 262144
net.core.wmem_default = 262144
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn.backlog=1024
```

- file-max

  表示进程（比如一个worker进程）可以同时打开的最大句柄数，这
  个参数直接限制最大并发连接数，需根据实际情况配置。

- tcp_tw_reuse

  设置为1，表示允许将TIME-WAIT状态的socket重新用于新的
  TCP连接，这对于服务器来说很有意义，因为服务器上总会有大量TIME-WAIT状态的连接。

- tcp_keepalive_time
  表示当keepalive启用时，TCP发送keepalive消息的频度。
  默认是2小时，若将其设置得小一些，可以更快地清理无效的连接。

- tcp_fin_timeout

  表示当服务器主动关闭连接时，socket保持在FIN-WAIT-2状
  态的最大时间。

- tcp_max_tw_buckets
  表示操作系统允许TIME_WAIT套接字数量的最大值，
  如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。该参数默认为
  180000，过多的TIME_WAIT套接字会使Web服务器变慢。

- tcp_max_syn_backlog

  表示TCP三次握手建立阶段接收SYN请求队列的最大
  长度，默认为1024，将其设置得大一些可以使出现Nginx繁忙来不及accept新连接的情况时，
  Linux不至于丢失客户端发起的连接请求。

- ip_local_port_range

  定义了在UDP和TCP连接中本地（不包括连接的远端）
  端口的取值范围。

- net.ipv4.tcp_rmem

  定义了TCP接收缓存（用于TCP接收滑动窗口）的最小
  值、默认值、最大值。

- net.ipv4.tcp_wmem

  定义了TCP发送缓存（用于TCP发送滑动窗口）的最小
  值、默认值、最大值。

- netdev_max_backlog

  当网卡接收数据包的速度大于内核处理的速度时，会有一个队列
  保存这些数据包。这个参数表示该队列的最大值。

- rmem_default

  表示内核套接字接收缓存区默认的大小。

- wmem_default

  表示内核套接字发送缓存区默认的大小。

- rmem_max

  表示内核套接字接收缓存区的最大大小。

- wmem_max

  表示内核套接字发送缓存区的最大大小。

## 进程

**nginx 的 master 进程主要用于读取和评估配置，并维护工作进度，提供命令行服务。**

**worker 进程是对请求进行实际处理，nginx 使用基于事件的模型和依赖操作系统的机制来高校地在工作进程之间分配请求。**

Nginx 时支持单进程<master 进程> 提供服务的，但是部署时都是使用一个 master 进程来管理多个 worker 进程，一般情况，worker 进程的数量与服务器上的 CPU 核心数相等。

worker 进程之间通过共享内存、原子操作等一些进程间通信机制来实现负载均衡等功能。

master 进程需要拥有较大的权限，通常会用 root 用户启动。

多个 worker 进程处理请求不但可以提供服务的健壮性，可以重新利用现在的 SMP 多核架构，实现微观上真正的多核并发处理。worker 进程可以同时处理的请求数只受限于内存大小。

在架构设计上，不同 worker 进程之间处理并发请求时几乎没有同步锁的限制，worker 进程通常不会进入睡眠状态，nginx 上进程数与核心数相等时（最好每一个 worker 进程都绑定特定的CPU核心），**进程间切换的代价是最小的**。

**worker_processes 可以设置worker 进程数和用户，多个进程或用户可以存在多个 nginx.conf 文件。**

这样就可以启动两个 worker 进程。

### worker 进程绑定 cpu 核心

避免 cpu 在切换进程时产生性能损耗，可以将 worker 进程和 cpu 进程进行绑定，这样 worker 进程就可以更好的专注使用某个 cpu 核心上的缓存，减少切换不同 worker 带来的缓存失效。

查看 cpu 核心

```
cat /proc/cpuinfo| grep "cpu cores"| uniq
```

**worker_cpu_affinity**  多少核多少位，01 表示绑定到第一个核上面

```
worker_processes auto;
worker_cpu_affinity 01 10;
```

### 查看进程cpu 核的绑定情况

分为两种情况

```
ps -ef|grep nginx 查看到 master 进程还是 worker进程
pidof nginx 查看所有 nginx pid
taskset -p[c] pid 
pid 1385's current affinity mask: 1
-p 显示的是十进制
```

**taskset 可以绑定进程到核心上面去**，强啊。

这个对于现在的我比较难记。。。

```
ps -eo pid,args,psr | grep nginx   
																	PSR 显示核
 1385 nginx: master process ./ngi   1
 7409 nginx: worker process         0
 7410 nginx: worker process         1
 8745 grep --color=auto nginx       0
```



## cli

通过 kill 命令发送信号，配置错误不会提示，还是 nginx 命令比较好

### 启动

- 默认方式启动会读取默认路径下的配置文件(configure --conf-path 指定的文件)

- 手动指定配置文件 -c

- 指定安装目录的启动方式 -p 

- 另行制定全局配置项启动 -g

  /sbin/nginx -g "pid /xx/a.pid"

  **-g 参数的约束条件**

  1. 制定的配置项不能与默认路径下的 nginx.conf 中的配置项冲突
  2. 以 -g 方式启动，指定其他命令也要把 -g 参数带上，否则可能出现配置项不匹配的情形

### 停止

**-s stop 强制停止**， -s 参数告诉 nginx 程序向正在运行的 nginx 服务发送信号量，nginx 通过 nginx.pid 文件中得到 master 进程的进程ID，再向 运行中的 master 进程发送 **TERM** 信号来快速关闭服务。

kill -s SINGTERM/SINGINT PID(master) 和 -s stop 是一样的。

**快速停止 worker 进程和 master 进程收到信号后会立刻跳出循环，退出进程**。

优雅停止 -s quit  / kill -s **SIGQUIT** <nginx master PID>，首先会关闭监听端口，停止收到新的连接，把当前正在处理的连接全部处理完，最后再退出进程。

**优雅停止某个 worker 进程**，可以通过发送 WINCH 信号，kill -s SIGWINCH <nginx worker pid>

### 重读配置

**-s reload**，nginx 会先检查新的配置项是否有误，如果全部正确就优雅的关闭，重启nginx来达到目的，可以使用 kill 命令发送 **HUP** 信号来达到相同的效果。

**kill -s SIGHUP master_pid**

### 日志回滚

**-s reopen** 可以重新打开日志，可以把当前的日志文件改名或转移到其他目录备份，再重新打开时就会生成新的日志文件，防止日志过大。

Kill 命令发送 USR1 信号效果相同

**kill -s SINGUSR1 master_pid**

### 平滑升级

nginx 支持不重启服务完成新版本的平滑升级

1. 通知正在运行的旧版本 nginx 准备升级，通过向master 进程发送 USR2 信号 **kill -s SIGUSR2 master_pid**

   运行中的 nginx 会将 pid 文件重命名，将 logs/nginx.pid 重命令为 logs/nginx.pid.oldbin，这样新的 nginx 才有可能启动成功。

2. 启动新的 nginx ，可以使用任何一种启动方法，ps 命令这时可以发现新旧版本的 nginx 再同时运行。

3. 通过 kill 命令向旧版本的 master 进程发送 SIGQUIT 信号，以优雅的方式关闭旧版本的 nginx，后面就只有新版本的 nginx 服务运行了。

### help

```
[root@c1 sbin]# ./nginx -?
nginx version: nginx/1.20.2
Usage: nginx [-?hvVtTq] [-s signal] [-p prefix]
             [-e filename] [-c filename] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /opt/dev/nginx/)
  -e filename   : set error log file (default: logs/error.log)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file
```

## 配置语法

按快配置划分，块配置项由一个块配置项名和一对大括号组成。

**块配置项可以嵌套，内层块直接继承外层块，内外层块中的配置发生冲突时，以哪里配置为准，取决于解析这个配置项的模块。**

**配置项名必须是某一个模块想要处理的**，否则 nginx 会认为出现了非法的配置项名。

**分隔符是空格，每行配置的结尾需要加上分号，配置项值中包括语法括号，需要用单引号或双引号括住**。

```
events{

}
http{
  upstream backend{
  	xxx
  }
  gzip on;
  server {
  	
  }
}
```

