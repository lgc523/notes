---
title: "Sentinel Guard"
date: 2022-02-09T11:32:01+08:00
draft: true
toc: true
tags: 
  - java
---

## Install dashboard

> git@github.com:lgc523/Sentinel.git

```
<repositories>
      <repository>
            <id>alimaven</id>
            <name>aliyun maven</name>
            <url>https://maven.aliyun.com/nexus/content/repositories/central/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
         </repository>
</repositories>
```

```bash
mci -> jar
cat guard-up.sh
#!/bin/bash
echo 'start sentinel-dashboard ...'
nohup java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-guard -jar sentinel-dashboard.jar > sentinel-dashboard.log 2>&1 &
echo $! > /opt/sentinel-dashboard/sentinel-dashboard.pid

$ cat guard-k.sh
#!/bin/sh
PID=$(cat /opt/sentinel-dashboard/sentinel-dashboard.pid)
kill -9 $PID
echo 'wait sentinel-dashboard...'
```

