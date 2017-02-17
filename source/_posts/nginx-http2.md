---
title: nginx开启http/2协议
date: 2017-02-17 14:51:50
tags:
  - ubuntu
  - nginx
  - http2
---

# 软件版本

开启http2需要nginx及openssl新版本支持，我使用的是ubuntu16.04，默认的nginx支持http2

更新nginx到最新版
```bash
$ apt-get update
$ apt-get upgrade
$ apt-get install nginx
```

# 修改配置文件

## 添加80端口监听

配置nginx监听80端口，并将请求转发到https

```nginx
server {
        listen 80;
        server_name www.lcorange.cn;
        rewrite ^/(.*)$ https://$host$1 permanent;
}
```

## 添加443端口监听

```nginx
server {
        listen 443;
        server_name www.lcorange.cn;

        location / {
                root /home/lcorange/blog/public;
                index index.html;
                expires 1d;
        }
}
```

## 添加https支持

复制ssl服务提供商提供的crt和key文件到`/etc/nginx/ssl`下

```nginx
server {
        listen 443;
        server_name www.lcorange.cn;

        ssl on;
        ssl_certificate ssl/YOUR.crt;
        ssl_certificate_key ssl/YOUR.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;

        location / {
                root /home/lcorange/blog/public;
                index index.html;
                expires 1d;
        }
}
```

## 添加http2支持

在目录`/etc/nginx/ssl`下生成pem文件

```bash
$ cd /etc/nginx/ssl
$ openssl dhparam -out dhparam.pem 2048
```

修改nginx配置

```nginx
server {
        listen 443 ssl http2;
        server_name www.lcorange.cn;

        ssl on;
        ssl_certificate ssl/YOUR.crt;
        ssl_certificate_key ssl/YOUR.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        ssl_dhparam ssl/dhparam.pem

        location / {
                root /home/lcorange/blog/public;
                index index.html;
                expires 1d;
        }
}
```

# 重启nginx

```bash
$ service nginx restart
```

# 开启效果

在chrome的开发者工具中，network标签下即可查看http协议，若显示h2，即为配置成功

![](/images/2017_2_17_h2_demo.png)
