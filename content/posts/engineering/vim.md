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
Plugin 'Valloric/YouCompleteMe'
call vundle#end()
```

## vimrc

```
"制表符和缩进、插入删除
set tabstop=4
set shiftwidth=4
set backspace=2

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

## vim

- yum remove vim -y

- make distclean

- ```bash
  ./configure --prefix=/usr/local/vim  \
  --enable-pythoninterp=yes \
  --enable-python3interp=yes \
  --with-python-command=python \
  --with-python3-command=python3
  ```

- make && make install

- vim --version |grep python

------



## cmake

- wget -c https://github.com/Kitware/CMake/releases/download/v3.21.5/cmake-3.21.5.tar.gz
- tar xvf
- ./bootstrap
- gmake
- gmake install
- yum remove cmake
- ln -s cmake /usr/bin/cmake

## Python3

- tar xvf
- yum install zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel xz xz-devel libffi-devel
- ./configure --enable-shared --enable-optimizations  --prefix=/usr/local/python3 && make && make install
- ln -s /usr/local/python3/bin/python3 /usr/bin/python
- cp /usr/local/python3/lib/libpython3.8.so.1.0 /usr/lib
- cd /etc/ld.so.conf.d
- vi python3.conf
- Ldconfig
- python -V
- Make distclean



- java11

    

- YouCompleteMe

    - git clone https://github.com/Valloric/YouCompleteMe.git ~/.vim/bundle/YouCompleteMe
    - cd  YouCompleteMe 
    - git submodule update --init --recursive
    - python3 ./install.py --java-completer --verbose


