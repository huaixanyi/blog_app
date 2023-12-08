---
title: acme获取https证书
comments: true
aside: true
top_img: false
date: 2021-10-21 13:36:24
tags:
description: acme获取https证书
mathjax: 
katex:
categories: net
cover: false
---
### 下载安装程序包，默认在用户文件夹下
```bash
curl https://get.acme.sh | sh
alias acme.sh=~/.acme.sh/acme.sh
echo 'alias acme.sh=~/.acme.sh/acme.sh' >>/etc/profile
```

### 生成证书

```bash
acme.sh --issue --nginx -d *.huaxianyi.com --webroot /data/wwwroot/hxy
```

#### 错误1
```bash
Can not find nginx conf.
```
##### 解决：使用nginx，安装时需要指定模块 `--conf-path` ，否则生成证书会报错`Can not find nginx conf.`
```bash
./configure --prefix=/usr/local/nginx \
--user=www --group=www \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-http_realip_module \
--with-http_ssl_module \
--with-stream_ssl_module --with-stream_realip_module \
--with-http_v2_module \
--with-stream
```

#### 错误2
```bash
Can not find conf file for domain
```
##### 解决：nginx配置文件需要配置生成证书时的域名代理信息
```bash
server {
listen       80;
server_name  www.xxx.com xxx.com blog.xxx,com;

        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
```

#### 成功会有证书路径
```bash
[Thu Oct 21 03:33:43 EDT 2021] Your cert is in: /root/.acme.sh/www.huaxianyi.com/www.huaxianyi.com.cer
[Thu Oct 21 03:33:43 EDT 2021] Your cert key is in: /root/.acme.sh/www.huaxianyi.com/www.huaxianyi.com.key
[Thu Oct 21 03:33:43 EDT 2021] The intermediate CA cert is in: /root/.acme.sh/www.huaxianyi.com/ca.cer
[Thu Oct 21 03:33:43 EDT 2021] And the full chain certs is there: /root/.acme.sh/www.huaxianyi.com/fullchain.cer
```

### 复制证书到nginx，方便管理
```bash
mkdir /usr/local/nginx/ssl_cert/
cp /root/.acme.sh/www.huaxianyi.com/fullchain.cer /usr/local/nginx/ssl_cert/huaxianyi.cer
cp /root/.acme.sh/www.huaxianyi.com/www.huaxianyi.com.key /usr/local/nginx/ssl_cert/
```

### 配置nginx配置文件
```bash
server {
		listen 80;
		listen 443 ssl;
		server_name www.huaxianyi.com huaxianyi.com;
		ssl_certificate /usr/local/nginx/ssl_cert/huaxianyi.cer;
		ssl_certificate_key /usr/local/nginx/ssl_cert/www.huaxianyi.com.key;
		
		ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
		ssl_ciphers ECDHE-RSA-AES256-SHA384:AES256-SHA256:RC4:HIGH:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!AESGCM;
		ssl_prefer_server_ciphers on;
		ssl_session_cache shared:SSL:10m;
		ssl_session_timeout 10m;
		
		location / {
			proxy_redirect off;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_pass http://127.0.0.1:4000;
		}
	}
```

### 重启nginx
```bash
nginx -s reload
```

# Have fun ^_^
---