---
title: "Centos8"
date: 2021-08-11T01:15:15+08:00
draft: true
author: "spider"
toc: getRuntime
images:
tags:
  - centos
---

## 1.mongodb

```shell
vi /etc/yum.repos.d/mongodb-org-4.4.repo

[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
```

```
yum install mongodb-org
systemctl enable mongodb --now
# configure file /etc/mongod.conf
# fix
security:
  authorization: enabled
```

```
#用户管理
db.version()
mongo --quiet "mongodb://${MONGO_HOST}:${MONGO_PORT}"
use db
db.createUser({user:"${MONGO_USERNAME}",pwd:"${MONGO_PASSWORD}",roles:["dbOwner"]})
dn.auth("username","password")
db.getUsers()
show users
db.system.users.find().pretty()

```

```
db.createUser(
  {
    user: "xxx",
    pwd: "xxx",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
db.createUser({user:"xxx",pwd:"xxx",roles:["dbOwner"]})
```

## 2.bottom

```
sudo yum install epel-release
sudo yum install snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
sudo snap install bottom
```

## 3.zsh

```
echo $shell
yum install -y zsh
chsh /bin/zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
