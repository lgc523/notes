``记录读书笔记``
### 生成静态/调试

```text
hugo serve -D -t
```
### 发布任务

```shell
echo "loading ..."
cd /opt/git/notes
git stash
git pull
echo "hugo generate static public ..."
hugo -D
echo "copy data ..."
rm -fr /opt/dev/nginx/html/public
mv public /opt/dev/nginx/html/
echo "reload nginx ..."
/opt/dev/nginx/sbin -s reload

#crontab -e
0 */1 * * * bash /opt/dev/update.sh > /opt/dev/update.log 2>&1 &
#reload crond service
systemctl restart crond
```

