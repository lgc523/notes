---
title: "Git"
date: 2022-02-08T23:40:45+08:00
draft: true
toc: true
tags: 
  - git
---

记录CVS 历史和一些命令。

```
alias gl="git log --graph --pretty=format:'%Cred%h%Creset - %C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
git config --global user.name
git config --global user.email
git config --global core.editor "vim"
git config --global color.ui true
git config --global core.quotepath false
cat ~/.gitconfig
git config -l
```

## git alias config

## git stash

``保存和恢复工作进度，强制重置暂存区和工作区``

- git stash 
- git stash [  save  [--patch]  [-k| --[no-]  keep-index ]  [ -q|--quiet ]  [<message>]
  - --patch 显示工作区和 HEAD 的差异
  - -k | --keep-index 保存进度后不会将暂存区重置
- git stash list
- git stash pop [--index] [stash]
  - 恢复最新保存的工作进度，并从工作进度列表中删除
  - stash 进度编号
  - --index 尝试恢复暂存区
- git stash apply  [--index]  [<stash>]
  - 不删除恢复的进度
- git  stash drop
  - 删除一个存储的进度，默认最新的进度
- git stash clear
  - 删除所有存储的进度
- git stash branch <branchname> <stash>
  - 基于进度创建分支
