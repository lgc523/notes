# My reading notes

### 生成静态/调试

```text
1.hugo
2.hugo serve -D -t hermit
```
### 定时发布脚本

```shell
echo "loading ..."
cd /opt/goProjects/notes
git stash
git pull
echo "hugo generate static public ..."
hugo -D
echo "copy data ..."
rm -fr /opt/nginx/html/public
mv public /opt/nginx/html/
echo "reload nginx ..."
/opt/nginx/sbin/nginx -s reload

#crontab -e
0 */1 * * * bash /opt/shells/pull.sh > /opt/shells/pull.log 2>&1 &
#reload crond service
systemctl restart crond
```

