---
title: "MySQL Install"
date: 2021-08-01T17:59:44+08:00
draft: true
author: "spider"
toc: true
tags:
  - mysql
---



- rpm -qa|grep mysql
- sudo yum install -y wget 
- wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
- sudo yum -y  localinstall mysql80-community-release-el7-3.noarch.rpm 
- sudo yum install -y yum-utils
- yum repolist enabled | grep 'mysql.*-community.*'
- yum repolist all | grep mysql
- 选择版本
- sudo yum-config-manager ----disable mysql80-community 禁用
- sudo yum-config-manager ----enable mysql57-community 启用5.7
- sudo yum install -y mysql-server
- systemctl enable --now mysqld
- grep 'temporary password' /var/log/mysqld.log
- alter user 'root'@'localhost' identified by 'xxx';
- create user 'x'@'y' identity by 'pass';
- grant all privileges on db_*.table_* to 'xxx'@'%';
- flush privileges;

select user,host,plugin from user;
