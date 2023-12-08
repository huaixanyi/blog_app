---
title: mwget（linux）多线程下载
comments: true
aside: true
top_img: false
date: 2021-10-21 13:36:24
tags:
description: （linux）多线程下载
mathjax:
katex:
categories:
cover: false
---

### 通过wget获取mwget程序包并解压
```bash
wget http://jaist.dl.sourceforge.net/project/kmphpfm/mwget/0.1/mwget_0.1.0.orig.tar.bz2
tar -jxvf mwget_0.1.0.orig.tar.bz2
cd mwget_0.1.0.orig
./configure
make && make install
```

### 报错1
```bash
[root@admired-beep-1 mwget]# tar -xjvf mwget_0.1.0.orig.tar.bz2
tar (child): bzip2: Cannot exec: No such file or directory
tar (child): Error is not recoverable: exiting now
tar: Child returned status 2
tar: Error is not recoverable: exiting now
```
#### bz2依赖程序包，安装
```bash
yum install bzip2
```

### 报错2
```bash
configure: error: Your intltool is too old.  You need intltool 0.35.0 or later.
```
#### 安装最新版本
```bash
yum install intltool
```

### 使用
```bash
mwget [URL]          // 默认开4个线程
mwget -n 10 [URL]   // 10个线程下载
```

# Have fun ^_^
---