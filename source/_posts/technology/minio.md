---
title: 对象存储服务器Minio
comments: true
aside: true
top_img: false
date: 2020-11-18 11:33:24
tags: 
- Minio
description: 对象存储服务器 Minio
mathjax:
katex:
categories: 
- technology
cover: http://42.192.149.99:9999/down/YK6LeoksZfw5?fname=/360wallpaper.jpg
top: true
video: true
---

> 官方介绍：
> MinIO 是一个基于Apache License v2.0开源协议的对象存储服务。它兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。
> MinIO是一个非常轻量的服务,可以很简单的和其他应用的结合，类似 NodeJS, Redis 或者 MySQL。
> 相对于FastDFS,部署简单,有UI界面,MinIO号称是世界上速度最快的对象存储服务器

### 本文使用系统CentOS Linux 7 (Core) x86_64
---
本文是Minio的简单使用，更多相关文档请至Minio官网查阅
[Minio 官方文档](https://docs.min.io/).

### 下载二进制文件
---
> GNU/Linux [Minio 官方安装文档](https://min.io/download#/linux).

```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
```

### 启动服务
---
```bash
chmod +x minio
MINIO_ACCESS_KEY=minioadmin MINIO_SECRET_KEY=minioadmin ./minio server /mnt/data
```
如果想要换验证密钥可将 minioadmin 值替换为自己想要的

### 使用MinIO浏览器进行验证
---
安装后使用浏览器访问 `http://127.0.0.1:9000` , 如果可以访问，则表示minio已经安装成功。
![minio_login_img](https://img.huaxianyi.com/huaxianyi/minio_login.png)
```bash
用户名密码
Access Key: minioadmin
Secret Key: minioadmin
```

### 外网永久分享链接
---
Minio分享链接最多只有7天的有效期
minio提供了一个客户端工具。可以直接对minio server进行配置，将指定桶设置为公共永久可下载。

|  系统   | 系统内核  | URL  |
|  ----  | ----  | ----  |
| GNU/Linux  | 64-bit Intel | https://dl.min.io/client/mc/release/linux-amd64/mc |
|   | 64-bit PPC | https://dl.min.io/client/mc/release/linux-amd64/mc |

```bash
https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
```

>Shell别名 可以添加shell别名来覆盖默认的Unix工具命令。

```bash
 alias ls='mc ls'
 alias cp='mc cp'
 alias cat='mc cat'
 alias mkdir='mc mb'
 alias pipe='mc pipe'
 alias find='mc find'
```

> 从MinIO服务获得URL、access key和secret key。
> 参数解析：`--api` API签名是可选参数，默认情况下，它被设置为"S3v4"。

```bash
mc config host add minio http://${ip}:{port} ${Access Key} ${Secret Key} --api s3v4
```

> 示例：

```bash
mc config host add minio http://192.168.1.51 BKIKJAA5BMMU2RHO6IBB V7f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12 --api s3v4
```

> 验证：mc将所有的配置信息都存储在~/.mc/config.json文件中。

```bash
cat ~/.mc/config.json
```

![minio_mc_yanzheng](https://img.huaxianyi.com/huaxianyi/minio_mc_yanzheng.png)

> mc预先配置了云存储服务URL：  `https://play.min.io`  ，别名“play”。它是一个用于研发和测试的MinIO服务。如果想测试Amazon S3,你可以将“play”替换为“s3”。
> 自己服务的别名叫  `minio`  ,具体可在  `~/.mc/config.json`  文件查看

```bash
## 示例:列出`https://play.min.io`上的所有存储桶。
mc ls play
[2016-03-22 19:47:48 PDT]     0B my-bucketname/
[2016-03-22 22:01:07 PDT]     0B mytestbucket/
[2016-03-22 20:04:39 PDT]     0B mybucketname/
[2016-01-28 17:23:11 PST]     0B newbucket/
[2016-03-20 09:08:36 PDT]     0B s3git-test/
```

#### policy命令 - 管理存储桶策略
---

```bash
用法：
  mc policy [FLAGS] PERMISSION TARGET
  mc policy [FLAGS] TARGET
  mc policy list [FLAGS] TARGET

PERMISSION:
  Allowed policies are: [none, download, upload, public].

FLAGS:
  --help, -h                       显示帮助。
```

> 此处我们选择 `public` 策略。

```bash
mc policy set public minio/blog
Access permission for `minio/blog` is set to `public`
```

> 接下来我们就可以永久访问啦 -> `https://www.example.com/bucket/example.png`

# Have fun ^_^ 
---