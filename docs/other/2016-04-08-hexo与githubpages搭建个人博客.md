---
title: hexo与githubpages搭建个人博客
date: 2016-04-08 21:45:15
tags: 
 - hexo
categories: 
 - other
---

## 软件环境
- 操作系统：windows
- node.js
- hexo
- github

## 原理简介
这所以能搭建这样一个免费的博客，是因为我们使用了Github的Pages功能，它提供了免费的空间来可以展示你的网站。而hexo一个博客框架提供了对博客相关一系列功能。有了以上两个前提我们就可以在本地用hexo写博客再将hexo生成的html上传到github，这样就完成了博客的搭建。

## 大致步骤
1.  安装git
2.  安装node.js
3.  安装hexo
6.  配置hexo  
7.  安装next主题
8.  配置next

git与node.js是安装hexo的前提。

##  安装Hexo\
可参考[hexo官方文档](https://hexo.io/zh-cn/docs/index.html)
git和node.js均可以windows下安装。
node安装成功可以使用``node -v``判断。
hexo安装成功可以使用``hexo -v``判断。
鼠标右键Git Bash使用如下命令安装hexo

    npm install -g hexo-cli

新建一个目录，比如这里就用hexo_dir右键这个目录 Git Bash，使用如下命令：

    hexo init <dir>

通过命令``hexo server``使用如下命令就可以在[连接](http://localhost:4000)查看到其首页了。
以上hexo就已经安装完成了。

## 配置github
新建repository命名为``<username>.github.io``

## 初步配置Hexo
hexo的配置文件在hexo_dir/_config.yml在此我使用的也是在网上找到的资源略作修改
```
## 配置信息：后必须有一个空格
## Site
title: Charlotte's Blog          ## 网站标题
subtitle: 随心 随笔             ## 简介
description: 随心 随笔          ## 描述
author: Charlotte                ## 网站作者
language: default         ## 网站语言
email: yhw1813@gmail.com
timezone: Asia/Hong_Kong        #时区

## Deployment
### Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git     ## 部署的位置  写git
  repository: git@github.com:CharlotteFly/charlottefly.github.io.git
  ## 这里填写项目的地址 git@github.com:Git用户名/Git用户名.github.io.git 参照上面
  branch: master    ## 分支选择 master
```
关于deployment部分可参考[deployment文档](https://hexo.io/docs/deployment.html)

安装发布到git所使用到的插件：

    npm install hexo-deployer-git --save ## 安装部署所需要的插件

##  安装主题
可参考[next主题官网](http://theme-next.iissnan.com/)

首先下载主题，在hexo_dir 右键Git Bash

``` 
##我这里使用的是next主题
git clone https://github.com/iissnan/hexo-theme-next.git themes/next
```

配置主题_config.yml修改：
```
theme: next
```

##  添加多说评论
在next主题已经添加了多说代码只需要指定多说的duoshuo_shortname就可以了
```
#多说评论
duoshuo_shortname: your_short_name
```
##  hexo分类与标签
默认next是不开启分类和标签的需要在主题目录(也就是hexo_dir/themes/next)下的_config.yml自行修改
```
menu:
  home: /
  categories: /categories
  #about: /about
  archives: /archives
  tags: /tags
  #commonweal: /404.html
```
但是这样还不行，还需要通过如下命令添加
```
hexo new page "tags"
nexo new page "categories"
```

至此，全部配置就完成了，通过下面的命令上传到github上吧。
```
hexo clean && hexo generate && deploy ## hexo clean && hexo d -g
```

## 相关文档
*  hexo:[https://hexo.io/zh-cn/docs]
*  next:[http://theme-next.iissnan.com/getting-started.html]