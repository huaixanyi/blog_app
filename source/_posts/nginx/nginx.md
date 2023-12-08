---
title: nginx安装
comments: true
aside: true
top_img: false
date: 2021-10-21 13:36:24
tags:
description: nginx安装
mathjax:
katex:
categories:
cover: false
---
### 安装依赖包
```bash
yum install gcc-c++  
yum install pcre pcre-devel  
yum install zlib zlib-devel  
yum install openssl openssl-devel 

/usr/sbin/groupadd -f www
/usr/sbin/useradd -g www www
```

### 获取nginx程序包并解压
#### 官方地址 https://nginx.org/download/
```bash
wget https://nginx.org/download/nginx-1.19.9.tar.gz

tar -zxvf nginx-1.19.9.tar.gz

cd nginx-1.19.9

./configure --prefix=/usr/local/nginx \
--user=www --group=www \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-http_realip_module \
--with-http_ssl_module \
--with-stream_ssl_module --with-stream_realip_module \
--with-http_v2_module \
--with-stream

make && make install
```

# Have fun ^_^
---