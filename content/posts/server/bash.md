---
title: "Bash"
date: 2021-11-02T23:36:35+08:00
draft: true
toc: true
images:
tags: 
  - server
---

## 系统的合法shell

``cat /etc/shells``

``` 
/bin/sh  被下面这个代替
/bin/bash 默认
/usr/bin/sh 
/usr/bin/bash
/bin/tcsh 整合 C shell
/bin/csh 被上面替换
```

## 查看、切换

`` echo $SHELL``

``cash -s /bin/zsh``

用户登陆，OS会给一个shell，登陆取得的shell记录在 **/etc/passwd** 

```
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
```

## history

很多Linux发行版，默认的命令记录条目为1000,记录在 **~/.bash_history**

``wc -l ~/.bash_history``  查看有2373行 。。。

## 内置命令判断

man page 查看文档

**type 命令重要在找出执行文件**

**type [-tpa] name**  

- 不加任何参数 显示时内部命令还是外部命令
- -t
  - file 表示外部命令
  - alias 别名
  - builtin 命令为bash内置命令
- -p 如果是外部命令才显示完整文件名
- -a  根据PATH 定义的路径中，将含有 name 的命令都列出来，包含alias

```
[root@c1 ~]# type gg
gg 是 `git push' 的别名
[root@c1 ~]# type -t gg
alias
[root@c1 ~]# type -p gg
[root@c1 ~]# type -a gg
gg 是 `git push' 的别名
[root@c1 ~]# type -t ll
alias
[root@c1 ~]# type -a ll
ll 是 `ls -alih' 的别名
[root@c1 ~]# type -p ll
[root@c1 ~]# type t ls
-bash: type: t: 未找到
ls 是 `ls --color=auto' 的别名
```

## 命令执行/编辑

长命令 使用 反斜杠 **\ENTER** 换行处理 

**删除命令 向前 ctrl + u ,向后 ctrl + k**

命令串中光标移动 ctrl + a，ctrl + e ，这个命令总是习惯的在vim的也用，。。。

## 变量设置规则

- 变量和内容之间用 = 连接，两边不能有空格

- 变量名称只能是英文字母与数字，不能以数字开头

- 变量内容有空格可以用 "  '   括起来

- **双引号内的特殊字符可以保留原本特性** la=" lang is $LANG"

-  反单引号 ` 包裹 command  或者 $ ( command ) 可以引用命令 

  - ver=`uname -r` echo $ver
  - ve=$(uname -r) echo $ve
  - **` 包裹的命令会先被执行，执行结果作为外部的输入信息** 

- 扩增变量

  ``PATH=$PATH:/usr/local/go/bin``

  ``name=${name}yes``

  ``name="$name"no``

- export

  变量需要在其他子程序中执行，export 使变量变成环境变量

- 取消变量 **unset** key

- 大写字符为系统默认变量

- 父进程自定义的变量无法在子进程内使用，export 设置为环境变量可以

## delcare

**声明**变量，不加参数显示所有的变量和函数

**bash 默认变量类型是字符串，bash 环境数值运算默认最多达成整数形态**

```
declare [+/-] [rxi] [变量名=设置值]
```

- +/"-"  "-" 设置变量，+ 取消变量属性
- -f 仅显示函数
- r 将变量设置为只读，只读后只能只读，无法再操作
- x 指定的变量会成为环境变量  
- i  设置值，可以是数值、字符串、运算式
- p 显示声明的类型

```
declare -x -a arr='([0]="l" [1]="g" [2]="c")' 设置数组
declare -p arr -> declare -arx arr='([0]="l" [1]="g" [2]="c")'
echo $arr -> l
echo ${arr[*]} -> l g c
declare -i sum=2+3/5   ->2
declare -i sum=(2+3)/5 -> 1
```

## env

env ，environment ，可以列出来所有的环境变量，主要包括当前的用户和所处的环境信息。

-  HISTSIZE 就表明了可以记录的记录数目

- Random 大多数发行版会有随机数生成器，**/dev/random ,可以通过 $RANDOM 获取随机数**，介于 0 ~ 32767

  获取 0-9 的数值，**declare -i number=$RANDOM*10/32768; echo $number**

## set

显示 环境变量 + 其他在bash内的变量

set ｜grep arr ，可以看到上吗声明的环境变量

## PS1

**提示字符**

``'[\u@\H \t {\w} {\#}]\$'``

提示字符，主动读取的值，可以自定义显示的值，时间、账户、目录

## $$

``echo $$`` 输入shell 的 PID

## ？

**上个指令的返回值**，**成功执行指令返回0，非0 表示执行错误**，可以通过 ``echo $?`` 来判断上一个命令是否执行成功

## export

**自定义变量转成环境变量**，~/.bash_porfile 里面的export 也是 declare -x 

在终端里面输入 bash，打开的是新的终端，是子进程，原来的终端就是暂停的状态，通过 ``echo $$`` 可以看到是不同的PID。

**子进程仅会继承父进程的环境变量，不会继承父进程的自定义变量。**

## locale

影响显示结果的语系变量。

``locale -a ``  语系文件 /usr/lib/locale 

locale  定义了一些单位、方式、习惯的格式 (12个)

格式文件存储在 /usr/share/i18n/locales

LC_ALL>LC*>LANG ，LC_[12]默认是LANG 。read

## Range

子进程可以使用父进程的环境变量，是因为子进程可以将父shell的环境变量所在的内存区域导入自己的环境变量区块中。

## read

读取键盘输入

```
read [-pt] variable
-p 提示字符
-t 等待时间
read -p "input your name: " -t 5 name
```

## ulimit

限制用户的的系统资源，用户登陆后加载文件生效

**/etc/security/limits.conf** 

```
#This file sets the resource limits for the users logged in via PAM.
#It does not affect resource limits of the system services.
```

``ulimit [-SHacdeilmnpqrstuvx] [配额]``

```
-H hard limit 严格设置不能超过
-S soft limit 警告设置，可以超过，超过有警告，要小于硬限制
-a 显示所有限制配额
-c 内核文件最大容量 程序发生错误，系统将程序的内存信息写成文件（core file）
-f 此shell 可以建立的最大文件容量(一般设置为2GB)，单位为 Kbytes 区块，只能向下调，不能向上调
-d 程序可使用的最大段内存容量(segment)
-l 可用于锁定的内存量
-t 可使用的最大的 CPU 时间,单位为秒
-u 用户可以使用的最大进程数量
-m 可使用的内存大小，单位kb
-n 同一时间最多可开启的文件数
-p 缓冲区大小，管道缓冲区的大小，单位512 Bytes
-s stack size kBytes
-v 使用的虚拟内存上限，单位kb
-i pending signals 可以被挂起/阻塞的最大信号数量
-e scheduling priority，进程优先级
```



```
[root@c1 15:35:06 ~ 21]#ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31194
max locked memory       (kbytes, -l) unlimited
max memory size         (kbytes, -m) unlimited
open files                      (-n) 100001
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0  # range[0,MAX_RT_PRIO-1]CFS
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 31194
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

## nice

进程优先级，range[-20,19]，数字越大，优先级越低，默认为0，ps、top 

PR = NI + 20，为内核的实际优先级

Nice [-n val] command 可以在启动时设置优先级

renice  [val] PID 可以在运行时调整，进程、用户、用户组

可以看到几个优先级比较高的进程 lru-add-drain，netns

**非root 用户只能往下调**

## 变量的替换、删除

