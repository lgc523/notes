---
title: "Nginx"
date: 2021-08-03T22:26:02+08:00
draft: true
author: "spider"
toc: true
tags:
  - middle-aware
  - nginx
---
#深入理解 Nginx

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

网站文件存放默认目录：/usr/share/nginx/html
网站默认站点配置：/etc/nginx/conf.d/default.conf
自定义Nginx站点配置文件存放目录：/etc/nginx/conf.d/
Nginx全局配置：/etc/nginx/nginx.conf
Nginx指定配置文件启动：nginx -c nginx.conf
PID目录：/var/run/nginx.pid
错误日志：/var/log/nginx/error.log
访问日志：/var/log/nginx/access.log
