---
date: 2018-08-11 14:27:00
status: public
title: shadowsocks服务端服务安装
categories: 
 - other
---
## Shadowsocks服务端服务安装


> 环境：centos

### 安装

安装python pip环境

```shell
sudo yum install python-setuptools && easy_install pip
```

通过pip安装shadowsocks

```shell
sudo pip install shadowsocks
```



### 配置

```shell
sudo vi /etc/shadowsocks.json
```

```json
{
  "server":"0.0.0.0",
  "local_address": "127.0.0.1",
  "local_port":1080,
  "server_port":my_server_port,
  "password":"my_password",
  "timeout":300,
  "method":"aes-256-cfb"
}
```



### 防火墙设置

```shell
firewall-cmd --permanent --add-port=my_server_port/tcp
firewall-cmd --permanent --add-port=my_server_port/udp
firewall-cmd --reload
```



### 服务启动

```shell
ssserver -c /etc/shadowsocks.json -d start
```





