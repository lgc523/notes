---
title: "Mac config"
date: 2021-08-20T23:39:36+08:00
draft: true
toc: true
images:
tags: 
  - engineering
---

最近突然就想把我的笔记本重置一下，就记录一下mac上面一些app的安装

### 重置m1

之前重置 mac 关机后 command + R 长按就可以到 recovery 界面，之前的 m1 Mac mini 我也忘记怎么重置了，m1 macbookpro

重置恢复不太一样，但是更简单了。

首先关机，长按电源键知道提示设置界面，磁盘管理-抹掉磁盘，紧接着需要 打开食用工具中的【终端】

输入 【resetpassword】 再进行重新安装 bigsur

### golang

```
export GO111MODULE=on
export GOROOT=/usr/local/go
export GOPROXY=https://goproxy.cn,direct
export GOPATH=/Users/lgc/Documents/golangProjects
export GOBIN=$GOPATH/bin
```

### hugo

```
git clone https://github.com/gohugoio/hugo.git
cd hugo
go install
cp $GOBIN/hugo /usr/local/bin
```

### iterm2-sz

```
brew install lrzsz
cd /usr/local/bin
vi iterm2-recv-zmodem.sh
vi iterm2-send-zmodem.sh
chmod 777 iterm2-*
brew info lrzsz 
cp sz rz -> /usr/local/bin

#config iterm2  profiles->advanced->triggers

Regular expression: rz waiting to receive.\*\*B0100
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-send-zmodem.sh
 
Regular expression: \*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
 
```

```
iterm2-recv-zmodem.sh
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required 
# Remainder of script public domain
 
osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi
 
if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    cd "$FILE"
    /usr/local/bin/rz -E -e -b
    sleep 1
    echo
    echo
    echo \# Sent \-\> $FILE
fi
```

```
iterm2-send-zmodem.sh
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required 
# Remainder of script public domain
 
osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi
if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    /usr/local/bin/sz "$FILE" -e -b
    sleep 1
    echo
    echo \# Received $FILE
fi 
```



### homebrew

```
/bin/bash -c "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install.sh)"
```

### zsh

```
brew install zsh
```

### oh-my-zsh

```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

```
vi ~/.zshrc
#配置插件
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
# 自动提示插件
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
# 语法高亮插件
git clone git://github.com/zsh-users/zsh-syntax-highlighting $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
#末尾
source ~/.bash_profile
source ~/.zshrc
```

### node

```
brew install node
#永久使用淘宝镜像代理
npm config set registry https://registry.npm.taobao.org
sudo npm install -g json
sudo npm install -g gtop
```

### /etc/hosts

```
127.0.0.1	localhost lgcs-MacBook-Pro.local
255.255.255.255	broadcasthost
::1             localhost lgcs-MacBook-Pro.local

151.101.130.132 rubygems.org
185.199.108.133 raw.githubusercontent.com
13.229.188.59 github.com
13.229.188.59 gist.github.com
172.67.3.188 v2ex.com

104.16.19.35 registry.npmjs.org
104.20.5.247 start.spring.io
```

### ~/.bash_profile

```
alias cb="cat ~/.bash_profile"
alias vb="vi ~/.bash_profile"
alias sb="source ~/.bash_profile"

alias mc="mvn clean"
alias mci="mvn clean install -T 1C -Dmaven.test.skip=true"
alias mmgg="mvn mybatis-generator:generate"
alias msbr="mvn spring-boot:run"

alias zks="/opt/zk361/bin/zkServer.sh status"
alias zkup="/opt/zk361/bin/zkServer.sh start"
alias zkk="/opt/zk361/bin/zkServer.sh stop"

alias check="git checkout"
alias br="git branch -av"
alias gp="git pull"
alias gg="git push"
alias gl='git log --date=format:"%Y-%m-%d %H:%M:%S" --pretty=format:"%h - %an, %ad: %s"'
alias gf='git remote update origin --prune'
alias gs="git status"
alias cl="git clone "
alias changelog="conventional-changelog -p angular -i CHANGELOG.md -s"
alias merge='git merge --no-ff'


alias ps='curl -H"Content-type:application/json"'
alias cl='curl -i -l'

alias c1="ssh -i ~/.ssh/xxxx.pem root@c1 -p xxxxx"
alias c2="ssh -i ~/.ssh/xxxx.pem root@c2 -p xxxxx"
alias c3="ssh -i ~/.ssh/xxxx.pem root@c3 -p xxxxx"


alias ll="ls -alih"
alias date='date "+%Y-%m-%d %H:%M:%S"'
#查看内存
alias mem='echo -e "\n$(top -l 1 | awk '/PhysMem/';)\n"'
alias pp="ifconfig |grep netmask"
#config for develop tools

#urlencode/decode
alias urldecode='python -c "import sys, urllib as ul; print ul.unquote_plus(sys.argv[1])"'
alias urlencode='python -c "import sys, urllib as ul; print ul.quote_plus(sys.argv[1])"'

export JAVA_HOME=/opt/jdk8u302-b08/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH:.

export GO111MODULE=on
export GOROOT=/usr/local/go
export GOPROXY=https://goproxy.cn,direct
export GOPATH=/Users/lgc/Documents/golangProjects
export GOBIN=$GOPATH/bin


export MAVEN_HOME=/opt/apache-maven-3.8.2
export PATH=$PATH:$MAVEN_HOME/bin
```

### ~/.ssh/config

```
host github.com
Hostname github.com
User lgc523
IdentityFile ~/.ssh/lgc

Host *
    ServerAliveInterval 60
```

### ~/.vimrc

```
设置tab 字符宽度
:set tabstop=4
:set shiftwidth=4
```



### maven

```
    <mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>central</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
```

ln -s $MAVEN_HOME/conf/settings.xml ~/.m2/settings.xml

### Source code pro

```
https://github.com/adobe-fonts/source-code-pro/releases/tag/variable-fonts
下载ttf 
```

### locale

```
重置后语言设置成了英文，也懒得改了，登陆服务器发现提示语言没有 change locale（UTF-8），字符集不对应
修改 ~/.zshrc
添加
export LC_ALL=en_US.UTF-8  
export LANG=en_US.UTF-8
这块在后面看 linux 私房菜再说
```

### Chrome plugin

- JSON-Handle
- Sourcegraph
- Talend API Tester - Free Edition
- Google Docs Offline

### Idea plugin

- Mybatisx
- RestfulTookit-fix
- POJO To Json
- SonarLint
- SQL Params Setter

### app

- File zilla 
- Snipaste
- AnotherRedisDesktop
- Subme Text
- Typora
- Wps
- BaiduNetdisk
- NeteaseMusic
- OneSwitch

### homebrew

- unar  

### command-line-tools

- zip	压缩文件

  ```
  zip -e -r dir xxx.zip 压缩文件夹
  zip -e xxx.zip file	  压缩单个文件
  ```

  
