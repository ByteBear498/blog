---
title: linux安装dropx
date: 2016-11-25 00:44:52
categories: 
 - other
---
本文内容来自dropbox官网，最新内容以官网为主，原文地址https://www.dropbox.com/zh_CN/install-linux
### 下载安装
32-bit:

`cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86" | tar xzf -`

64-bit:

`cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -`

### 运行
接着，从新建的 .dropbox-dist 文件夹运行 Dropbox 守护程序。

`~/.dropbox-dist/dropboxd`

### 关联
如果是首次在服务器上运行 Dropbox，系统会要求您将链接复制并粘贴到运行的浏览器中，以便创建一个新的帐户或将服务器附加到现有帐户上。操作完成后，系统会在您的主目录中创建 Dropbox 文件夹。下载这个 [Python][1] 脚本，通过命令行控制 Dropbox。为了方便访问，请在 PATH 中的任何地方放入此脚本的符号链接。


[1]:https://www.dropbox.com/download?dl=packages/dropbox.py