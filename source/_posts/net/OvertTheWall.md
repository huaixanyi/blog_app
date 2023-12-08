---
title: 科学上网之v2ray
comments: true
aside: true
top_img: false
date: 2020-11-18 17:35:20
tags: OvertTheWall
description: 科学上网之v2ray
mathjax:
katex:
categories: 服务器
cover: false
---
> 介绍：V2Ray是一个网络代理工具，目前支持Windows、Mac、Android、IOS、Linux等操作系统的使用。
> 在混淆能力、兼容性、速度上有着独到的优点。
* VPS
* v2ray
* 第三项

### 购买VPS
--- 
> 推荐购买 [搬瓦工VPS](https://bwh88.net/) 买最便宜的即可。 一年不到300 , 方便自己用 , 速度稳定 , 不吃亏。
> 本人用过Vultr, 不稳定, 速度快的地区不好找, 即便找到也容易被墙, 而且也不便宜。

![20201106180221](http://img.huaxianyi.com/huaxianyi/微信图片_20201106180221.png)

### 安装bbr加速系统
---
> 进入控制台页面，关闭服务器, 重新安装支持bbr加速的系统,即选项中系统名称带后缀  `-bbr`

![control](http://img.huaxianyi.com/huaxianyi/control.png)

![install_os](http://img.huaxianyi.com/huaxianyi/install_os.png)

### 连接远程服务器，并安装v2ray
---
```bash
bash <(curl -s -L https://git.io/v2ray.sh)
```

> 按最简单的来，回车，全部使用默认配置

![v2ray_install](http://img.huaxianyi.com/blog/v2ray_install.png)


### 获取v2ray url协议连接
---
> 安装完成，获取v2ray url协议连接

```
v2ray url
```

### 安装windows客户端
---
> v2rayN 是Windows平台上一款基于v2ray核心的简洁好用、功能强大的v2ray客户端，支持Vmess、Shadowsocks、Socks5等多种协议，也支持服务器订阅

[v2rayN下载](https://github.com/v2fly/v2ray-core/releases/tag/v4.32.0)

### 安装安卓客户端

[v2rayN安卓客户端](https://github.com/2dust/v2rayNG/releases/tag/1.4.11)

# 安装完成 Have fun ^_^ 
---
