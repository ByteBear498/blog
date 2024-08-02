---
title: ubuntu14.4-server控制台中文乱码问题解决
date: 2016-04-09 10:42:36
tags: 
 - ubuntu
categories: 
 - other
---
## 问题
今天在安装ubuntu14.4 server版本后，出现了乱码问题。
## 解决方法
修改/etc/default/local文件
```
LANG="en_US.UTF-8"
LANGUAGE="en_US:en"
```
## 问题原因分析
大概是我在安装系统的时候选择了中文，所以控制台想打印的中文字符，但解码时出现了问题。该解决方法也只是将中文还原成了英文并没有根本解决问题。PS:话说，其实还是看英文的比较习惯吧。