---
title: 基于nginx的acme免费证书方案
date: 2020-03-02 17:36:15
tags: 
 - nginx
 - acme
categories: 
 - other
---



为了保护用户信息使网站更加安全，需要给网站添加https协议。搭建一个https网站的前提是要先拥有一个**证书**，当然一般的证书是需要收费，文本提供一个免费的解决方案。



我们使用的github上开源的免费证书工具ACME.SH，网站地址：https://github.com/acmesh-official/acme.sh，

在其github主页上已经有了一些很好的入门说明。这里根据实际情况进行操作。



### 依赖安装

```shell
apt-get install cron socat -y
```




### 安装ACME.sh

```shell
curl https://get.acme.sh | sh 
```

安装成功后会提示`Install success!`，这个命令会将acme命令写到`batchrc里`，为了方便使用需要 ：

```shell
source ~/.bashrc 
```



### 申请证书

```shell
acme.sh --issue -d example.com --standalone
```

 生成的证书被放在 `/root/.acme.sh/`



### 复制证书到nginx

```shell
acme.sh --installcert -d freegoooovideos.ml \
        --key-file /etc/nginx/ssl/freegoooovideos.key \
        --fullchain-file /etc/nginx/ssl/fullchain.cer \
        --reloadcmd "systemctl force-reload nginx.service"  #或sudo service nginx force-reload

```

 `reloadcmd` 会记住让nginx重新加载的方式 ，这样证书更新的时候就可以让你nginx重新加载了。

这个命令不只是复制，它会把信息记录到本地中（`.acme/example.com/example.com.conf`），这样在更新证书的时候会自动将文件复制到容器中，并让其重新加载配置。



附：nginx的配置文件（/etc/nginx/nginx.conf 默认位置）

```nginx
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
  map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
  }
  server{
    listen 443 ssl;
    ssl on;
    proxy_redirect off;
    ssl_certificate       /etc/nginx/ssl/fullchain.cer;
    ssl_certificate_key   /etc/nginx/ssl/{example}.key;
    ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers           HIGH:!aNULL:!MD5;
    server_name           {exanoke.com};
    location /freevideos {
      access_log /var/log/nginx/access.log;
      proxy_pass http://127.0.0.1:4433;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
  }
}
```







