---
title: "Crontab"
date: 2021-08-17T00:43:43+08:00
draft: true
toc: true
images:
tags: 
  - 定时任务
  - linux
---

## linux cron

```
常用命令
crontab [-u username] //默认是当前用户的 crontab
	-e 编辑工作表
	-l 列处工作表里面的命令
	-r 删除工作表
* 取值范围内的所有数组
/ 每过多少数字
- 从 X 到 Z
, 散列数字
```

```
3,15 8-11 */2  *  * myCommand 每隔两天的上午8点到11点 第三、第五分钟执行
3,15 8-11 * * 1 myCommand     每周一上午8点到11点的第3和第15分钟执行
0 23-7/1 * * * /etc/init.d/smb restart 每天的晚上十一点到期待呢 每小时重启
```



```
#1.负责调度各种管理和维护任务
[root@c3 cron]# cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

#2.目录下存放的是每个用户包括root的crontab任务，每个任务以创建者的名字命名
[root@c3 cron]# cat /var/spool/cron/root 
*/5 * * * * flock -xn /tmp/stargate.lock -c '/usr/local/qcloud/stargate/admin/start.sh > /dev/null 2>&1 &'
0 */1 * * * bash /opt/shells/pull.sh > /opt/shells/pull.log 2>&1 &

#3. /etc/cron.d/ 这个目录用来存放任何要执行的crontab文件或脚本。
#4. 还可以把脚本放在/etc/cron.hourly、/etc/cron.daily、/etc/cron.weekly、/etc/cron.monthly目录中，让它每小时/天/星期、月执行一次。
```

### Blog 目前每小时更新(-> gitlab ci/cd)

```shell
echo "loading ..."
cd /opt/goProjects/notes

git stash

git pull

echo "hugo generate static public ..."
/opt/goProjects/bin/hugo -D

echo "copy data ..."

rm -fr /opt/nginx/html/public
mv public /opt/nginx/html/

echo "reload nginx ..."

/opt/nginx/sbin/nginx -s reload
```

``0 */1 * * * bash /opt/shells/pull.sh > /opt/shells/pull.log 2>&1 &``

## ScheduledExecutorService

```

```

