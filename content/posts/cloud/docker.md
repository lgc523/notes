---
title: "Docker"
date: 2021-08-11T00:44:20+08:00
draft: true
author: "spider"
toc: true
tags:
  - cloud
  - docker
---

## install

``centos7``

```
1.yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-ce

2.yum list installed  | grep docker   
3.rm -rf /var/lib/docker rm -fr /etc/docker rm -fr ~/.docker
4.yum install -y yum-utils
5.yum-config-manager \
 --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
6.yum repolist
7.yum list docker-ce --showduplicates | sort -r
8.yum -y install docker-ce-19.03.8 docker-ce-cli-19.03.8 containerd.io

{
  "registry-mirrors": ["https://mirror.ccs.tencentyun.com"],
	"insecure-registries":["guangchang.tech:5000"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}

```

## image

```
docker [image] pull [url/]NAME[:TAG] 默认 TAG latest
镜像文件一般由若干层 layer 组成，id 是层唯一的 id(实际上完整的 id 包括 256 bit, 64 个二进制字符组成)
当不同景象包括相同的层时，本地仅存储了层的一份内容，减少了存储空间
```

### 1.下载命令

```
-a  --all-tags=true|false 是否获取仓库中的所有镜像，默认为否
--disable-content-trust 取消镜像的内容校验，默认为真
--registry-mirror=proxy_URL 镜像代理服务地址
```

### 2.查询镜像命令ls、tag、inspect

#### ls

```
docker images ｜ docker image ls 列出本地主机上已有镜像的基本信息
列出信息
来自于哪个仓库
镜像的标签，只是标记，不能够标识镜像内容
镜像的id 唯一标识镜像，两个镜像相同，实际指向了同一个镜像
创建时间 镜像最后的更新时间
镜像大小 逻辑提及大小

-a --all=true｜false 列出所有包括临时文件镜像文件，默认为否
--digests=true|false 列出镜像的数字摘要值，默认为否
-f --filter=[] 过滤列出的镜像。dangling=true 只显示没有被使用的镜像
--format="TEMPLATE": 控制输出格式
--no-trunc=true|false 对输出结果中太长的部分是否进行截断，默认为是
-q --quiet=true｜false 仅输出ID信息，默认为否
```

#### tag

```
为镜像添加标签|重命名
docker tag xxx:yyy newXXX:yyy
类似 ln 软链接
```

#### inspect

```
docker  inspect image 获取镜像的详细信息，包括制作者、适应架构、各层的数字摘要 返回 json 格式

docker  inspect -f {{"".Architecture"}} xxx:yyy
-s if the type is container
```

#### history

```
docker history xxx:yyyy 列出各层的创建信息
--no-trunc=false 输出完整命令
```

### 3.搜寻镜像

```
docker search [option] keyword
-f --filter filter:过滤输出内容
--format string: 格式化输出内容
--limit int 限制输出个数，默认25个
--no-trunc 不截断输出结果

docker search --filter=is-official=true nginx
docker search --filter=start=15 nginx --limit 15
```

### 4.删除和清理镜像

#### 标签删除

```
docker rmi / docker image rm
docker rmi IMAGE [IMAGE...]   IMAGE 可以为标签或id
-f -force 强制删除，即使有容器依赖
-no-prune 不要清理未带标签的父镜像
多个标签链接到同一个镜像，只是删除一个标签
```

#### 镜像id删除

```
docker rmi id
会先尝试删除所有指向该镜像的标签，然后删除该镜像文件本身
如果该镜像创建的容器存在时，镜像文件默认是无法被删除的
```

#### 镜像清理

```
docker image prune
-a -all 删除所有无用镜像，不光是临时镜像
-filter filter 只清理符合给定过滤器的镜像
-f -force 强制删除镜像，不进行提示确认
```

### 5.创建镜像

```
创建镜像方法主要有三种： 基于已有镜像的容器创建、基于本地模版导入、基于Dockerfile 创建
commit import build
```

#### 基于已有容器创建

```
docker [container] commit
docker [container] commit [options] container [repository] [:TAG]
-a --author="" 作者信息
-c --change=[] 提交的时候执行 Dockerfile 指令，包括 CMD|ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|VOLUME|WORKDIR
-m --message="" 提交信息
-p --pause=true 提交时暂定容器运行
```

#### 基于本地模版的导入

```
docker [image] import [options] file|URL| -[repository][:TAG]
```

#### 基于Dockerfile 创建

```dockerfile
Dockerfile 是一个文本文件，利用给定的指令描述基于某个父镜像创建新镜像的过程

FROM 基础镜像
MAINTAINER 维护者信息
RUN 操作 构建过程中执行
ADD copy 文件，会自动解压
COPY sourcedir/file dest_dir/file 拷贝文件，不会解压
WORKDIR 切换工作目录
VOLUME 目录挂载
EXPOSE 内部服务端口
CMD 执行 DockFile 中的命令 容器启动后执行
ENV 设置环境变量
```

### 6.存出/载入镜像 save/load

```
docker [image] save/load
```

- 存出镜像

  ```
  docker save -o outName xxx:yyy
  -o -output string 指定导出到指定文件
  ```

- 载入

  ```
  docker load -i xxx
  docker load < xxx
  -i -input string 指定文件中读入镜像内容
  ```

### 7.上传镜像

```
docker [image] push 上传镜像到仓库，默认上传到 Docker Hub 官方仓库，需要登陆，登陆后信心保存在～/.docker目录
docker [image] push NAME[:TAG] | [registry_host[:registry_port]/]NAME[:TAG]
```

## container

```
容器是镜像的一个运行实例，镜像是静态的只读文件，容器是带有运行时需要的可写文件层，容器的应用进程处于运行状态
```

### 1.创建容器

```
docker [container] create
docker run -it --privileged --name=容器名称 镜像名称:标签 /bin/bash #一次性交互 exit 退出容器
docker run -dt --name=容器名称 镜像名称:标签 cmd # detach
docker run -d -p 8080:80 --name Nginx nginx
登陆后台启动容器
docker exit -it 容器名称 /bin/bash
run 									创建容器
-i										运行容器
-d 										detach以后台 daemon 的方式运行
--name      					指定一个容器的名字,后面操作系统通过这个名字你在定位容器
-t										分配一个伪终端，启动后会进入命令行 Allocate a pseudo-TTY
-p 										端口映射
	-P 随机映射
	-p 指定
		hostport:containerport
		ip:hostport:container:port
		ip::conntainerport
		hostport:containerport:|udp
-v 										挂载 volume
-net									指定容器的网络连接类型 bridge / host / none / contriner:<name|id>
ping 114.114.114.114	启动容器执行的命令
--privileged 				 	真正的root权限
```

### 命令与运行模式选项

| -a --attach=[]               | 是否绑定到标准输入、输出和错误                               |
| ---------------------------- | ------------------------------------------------------------ |
| -d --detach=true\|false      | 是否在后台运行容器，默认为否                                 |
| --detach-keys=""             | 从attach模式退出的快捷键                                     |
| --entrypoint=""              | 镜像存在入口命令时，覆盖为新的命令                           |
| --expose=[]                  | 指定容器会暴露出来的端口或端口范围                           |
| --group-add=[]               | 运行容器的用户组                                             |
| -i --interactive=true\|false | 保持标准输入打开，默认false                                  |
| --ipc=""                     | 容器IPC命令空间，可以为其他容器或主机                        |
| --isolation="default"        | 容器使用的隔离机制                                           |
| --log-driver="json-file"     | 指定容器的日志驱动类型，可以为json-file、syslog、journals、self、f luentd、analogs或none |
| --log-opt=[]                 | 传递给日志驱动的选项                                         |
| --net-alias=[]               | 容器在网络中的别名                                           |
| -P --publish-all=true\|false | 通过NAT机制将容器标记暴露的端口自动应色号到本地主机的临时端口 |
| -p --publish=[]              | 指定如何映射到本地主机端口                                   |
| --pid=host                   | 容器的 PID 命名空间                                          |
|                              |                                                              |
|                              |                                                              |
|                              |                                                              |
|                              |                                                              |
|                              |                                                              |
|                              |                                                              |
|                              |                                                              |
|                              |                                                              |
|                              |                                                              |
|                              |                                                              |
|                              |                                                              |

### 2.启动容器

```
docker start container 命令来启动一个已经创建的容器
docker ps
docker run = docker create && start
```

```
docker run container xxx
1.检查本地是否存在指定的镜像，不存在就从公有仓库下载
2.利用镜像创建一个容器，并启动该容器
3.分配一个文件系统给容器，并在只读的镜像层外挂一层可读写层
4.从宿主机配置的网桥接口中桥接一个虚拟接口到容器中去
5.从网桥的地址池配置一个IP地址给容器
6.执行用户指定的应用程序
7.执行完毕后容器被自动终止
```

```
docker run -it xxx /bin/bash
-t 分配一个伪终端，并绑定到容器的标准输入上，
-i 让容器的标准输入保持打开
-d 守护态运行
exit 退出容器
```

### 3.查看容器日志

```
docker logs container
--details 打印详细信息
-f -follow 持续保持输出
--since string 输出从某个时间开始的日志
-n --tail string 输出最近的若干日志
-t -timestamps 显示时间戳信息
-until string 输出某个时间之前的日志

docker run -dit ubt /bin/bash -c "while true; do echo hello world;sleep 1;done"
```

### 4.列出容器

```
docker ps [option]
-a 显示所有容器，包括未运行的
-f 根据条件过滤
--format 指定返回值的模版文件
-l 显示最近创建的容器
-n 列出最近创建的n个容器
--no-trunc 不截断输出
-q 静默模式，只显示容器编号
-s 显示总文件大小
docker ps -a --filter 'exited=0'
docker ps --filter status=running
docker ps --filter status=paused
```



### 5.暂停容器

```
docker pause container 停止容器
docker unpause container 恢复运行
docker restart container 命令会将一个运行态的容器先终止，在重新启动
```

### 6.停止容器

```
docker stop container -t[--time[=n]]
stop 命令会先向容器发送 SIGTERM 信号，等待一段时间后（默认为10s），在发送 SIGKILL 信号来终止容器
docker ps -qa 查看所有container id
```

### 7.进入容器

```
使用 -d 参数，容器启动后会进入后台，需要进入容器进行操作，官方推荐使用 attach / exec 命令
```

- attach

  ```
  docker attach [--detach-keys[=]]] [--no-stdin] [--sig-proxy[=true]] container
  ---detach-keys[=[]] 指定退出 attach 模式的快捷键序列，默认是 CTRL -p CTRL -q
  --no-stdin=true|false 是否关闭默认标准输入，默认是保持打开
  --sig-proxy=true|false 是否代理收到的系统信号给应用进程，默认是true
  ```

- exec

  ```
  Docker 1.3.0 exec 可以直接在运行中的容器内执行任意命令
  docker exec [-d| --detach] [--detach-keys[=[]]] [-i|--interactive] [--privileged] [-t|--tty] [-u|--user[=USER]] container command args...
  -d --detach 在容器中后台执行命令
  --detach-keys="" 指定将容器切回后台的按键
  -e --env=[] 指令环境变量列表
  -i --interactive=true|false 打开标准输入接受用户输入命令，默认值false
  --privileged=true|false 是否给执行命令以高权限，默认值为false
  -t --tty=true｜false 分配伪终端，默认值为false
  -u --user="" 执行命令的用户名或ID
  eg:
  	docker exec -it xxx /bin/bash
  	-it 保持标准输入打开，并且分配一个伪终端
  ```

### 8.删除容器

```
docker rm [-f｜-force] [-l|--link] [-v|--volumes] container 删除处于终止或退出状态的容器
-f --force=false 是否强制执行并删除一个运行中的容器
-l --link=false 删除容器的链接，但保留容器
-v --volumes=false 删除容器挂载的数据卷
```

### 9.导入和导出容器

```
导出 导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态
docker export  [-o|--output[=""]] container
-o 指定导出的 tar 文件名
```

```
导入
docker import [-c｜--change[=[]]] [-m|--message[=MESSAGE]] file|URL|-[REPOSITORY[:TAG]]
用户可以通过 -c --change=[] 选项在导入的同时执行对容器进行修改的Dockerfile指令
```

#### 区别 import|load

```
import 导入一个容器快照到本地镜像库，容器快照文件将丢失所有的历史记录和元数据信息，仅保存容器当时的快照状态
load 镜像存储文件将保存完整记录，体积更大
从容器快照文件导入时可以重新指定标签等元数据信息
```

### 10.查看容器 inspect|top|stats

```
查看容器详情可以使用 docker container inspect [options] container
查看容器内进程 docker top xxx
查看统计信息 docker stats xxx
		-a --all 输出所有容器统计信息，默认仅在运行中
		-format string 格式化输出信息
		-no-stream 不持续输出，默认会自动更新持续实时结果
		-no-trunc 不截断输出信息
```

### 11.cp|diff|port|update

- ```
  复制文件 docker cp [options] container:src_path dest_path|-
  	-a -archive 打包模式，复制文件会带有原始的 uid/gid
  	-L -follow-link 跟随软连接，当原路径为软连接时，默认是只复制链接信息，使用该选项会复制链接的目标内容
  ```

- ```
  查看变更 docker diff xxx 查看容器内文件系统变更
  ```

- ```
  查看端口映射 docker port xxx [private_port[/proto]]
  ```

- ```
  更新配置 docker update xxx 更新运行时配置，主要是一些资源的限制份额
  -blkio-weight uint16 更新块 io 限制，10-1000 默认0，代表无限制
  -cpu-period int 限制 cpu 调度器 (completely fair scheduler) 使用时间，单位为毫秒，最小1000
  -cpu-quota int 限制cpu 调度器 cfs 配额，单位为毫秒，最小 1000
  -cpu--rt-period int 限制 cpu 调度的实时周期，单位为毫秒
  -cpu-rt-runtime int 限制 cpu 调度器的实时运行时，单位为毫秒
  -c -cpu-shares int 限制 cpu 使用份额
  -cpus decimal 限制 cpu 个数
  -cpuset-cpus string 允许使用 cpu 核
  -cpuset-mems string 允许使用的内存块
  -kernel-memory bytes 限制使用的内核内存
  -m -memory bytes 限制使用的内存
  -memory-reservation bytes 内存软限制
  -memory-swap bytes 内存加上缓存区的限制 -l 表示对缓冲区无限制
  -restart string 容器退出后的重启策略
  ```

## 仓库

### 搭建本地私有仓库

```
1.docker run -d -p 5000:5000 --restart=always  -v /data/docker_local_repo:/var/lib/registry --name registry registry
# /etc/docker/daemon.json
{
 "registry-mirrors":["https://mirror.ccs.tencentyun.com"],
 "insecure-registries":["guangchang.tech:5000"]
}
2.docker tag image:tag registry_host/username/name:tag 标记
3.docker push xxx 上传标记的镜像
4.curl ip:port/v2/_catalog
5.docker pull xxx/yyy:version
查看images versions
curl ip:port/xxx/tags/list
```



## 数据管理

```
管理数据的两种主要方式
1.数据卷    容器内数据直接映射到本地主机环境
2.数据卷容器 使用特定的容器维护数据卷
```

### 数据卷

```
数据卷是一个可供容器使用的特殊目录，将主机操作系统目录直接映射进容器，类似于linux中的mount行为
特性
1.数据卷可以在容器之间共享和重用，容器间传递数据将变得高效方便
2.对数据卷内数据的修改会立马生效，无论是容器内操作还是本地操作
3.对数据卷的更新不回影响镜像，解耦开应用和数据
4.卷会一直存在，知道没有容器使用，可以安全的卸载
```

#### 创建数据卷

```
volume 子命令管理数据卷
docker volume create -d local test
/var/lib/docker/volumes  #查看
docker volume 还支持 inspect ls prune rm
#
docker volume ls
docker volume inspect test
[
    {
        "CreatedAt": "2021-06-18T00:33:25+08:00",
        "Driver": "local", #使用的驱动程序，使用宿主机的本地存储
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/test/_data",#挂载点
        "Name": "test",
        "Options": {},
        "Scope": "local"
    }
]
#安装插件，通过vieux/sshfs 驱动把数据卷的存储在其他主机上
docker plugin install --grant-all-permissions vieux/sshfs
#指定远程主机的登陆用户名、密码和数据存放目录
docker volume create --driver vieux/sshfs \
-o sshcmd=user@ip:/remote_path\ #确保远程目录存在,否则启动报错
-o password=password \
mysshvolume

#启动容器挂载远程数据卷
docker run -id \
    --name testcon \
    --mount type=volume,volume-driver=vieux/sshfs,source=mysshvolume,target=/world \
    ubuntu /bin/bash
```

#### 绑定数据卷

```
可以在创建容器时将主机本地的任意路径挂载到容器内作为数据卷
docker  run 可以使用 -mount 选项使用数据卷
-mount 支持三种类型的数据卷
volume 普通数据卷 映射到主机 /var/lib/docker/volumes 路径下
bind	 绑定数据卷，映射到主机指定路径下
tmpfs  临时数据卷，只存在于内存中
```

```
docker run -d -P --name xxx --mount type=bind,source=/x,destination/y container
docker run -d -P --name xxx --mount type=bind,src=/x,dst=/y container
==
docker run -d -P --name xxx -v /src:/dest
本地路径必须是绝对路径，容器内路径可以是相对路径，目录不存在，docker 会自动创建
Docker 挂载数据卷默认的默认权限是 读写rw，可以通过ro指定为只读
docker run -d -P --name xxx -v /src:/dest:ro
如果直接挂载一个文件到容器，使用文件编辑工具，包括 vi、sed --in-place时候，可能会造成文件inode的改变
docker 1.10起，会导致报错误信息，推荐直接挂载文件所在的目录到容器内
```

## 数据卷容器

```
如果用户需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器
数据卷容器也是一个容器，目的是专门提供数据卷给其他容器挂载的
```

### 创建数据卷容器

```
docker run -it -v /dbdata --name container [command]
创建好的数据卷是停止状态，因为使用 --volumes-from 参数挂载数据卷的容器自己不需要保持在运行状态
#创建容器挂载数据卷

```

### 数据覆盖问题

```
挂载一个空的数据卷到容器中的一个非空目录中，那么这个目录下的文件会被复制到数据卷中
挂载一个非空的数据卷到容器中的一个目录中，容器中的目录会显示数据卷中的数据，如果原来容器中的目录又数据，这些原始数据会被隐藏掉
```
