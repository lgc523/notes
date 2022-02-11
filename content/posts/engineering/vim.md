---
title: "Vim"
date: 2022-02-11T13:46:19+08:00
draft: true
toc: true
tags: 
  - engineering
---

## color schema 

> https://vimcolorschemes.com/
>
> https://github.com/flazz/vim-colorschemes.git

## vundle

```
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim

set rtp+=~/.vim/bundle/Vundle.vim

call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
Plugin 'fatih/vim-go'
Plugin 'flazz/vim-colorschemes'
call vundle#end()
```

## vimrc

```
"制表符和缩进
set tabstop=4
set shiftwidth=4


set rtp+=~/.vim/bundle/Vundle.vim

call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
Plugin 'fatih/vim-go'
Plugin 'flazz/vim-colorschemes'
call vundle#end()


"光标高亮
set cursorline
"取消 vi 兼容
set nocompatible
"文件检测
filetype on
"状态行显示命令
set showcmd
"命令行高度
set cmdheight=3
"搜索忽略大小写
set ignorecase

"搜索高亮
set hlsearch
"逐字符高亮
set incsearch
"行号
set nu

"允许使用插件
filetype plugin on
filetype plugin indent on

"语法高亮
syntax on
set background=dark
colorscheme gruvbox
```

## Vim-go

```
:PluginInstall
:GoInstallBinaries
```



## linux

Upgrade vim 

```
git clone https://github.com/vim/vim.git
./configure --prefix=/usr/local/vim  --enable-pythoninterp=yes --enable-python3interp=yes --with-python-command=python --with-python3-command=python36

```



- cmake
- java11
- Python3
- YouCompleteMe
    - git remote set-url origin https://github.com/ycm-core/YouCompleteMe.git
    - git fetch
    - git submodule update --init --recursive
    - python3 install.py --all

