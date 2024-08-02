---
title: "使用GitHub Actions自动部署hugo到GitHub Pages"
date: 2021-05-05 14:16:16
categories: 
 - other
tags: 
 - blog
 - hugo
 - github actions
 - github pages
---

之前一直使用`Bitcron `来搭建人个博客，但是最近`Bitcron `的官网显示，未来`Bitcron `将升级到`FarBox 2.0`，22年底就不再支持`Bitcron `了，并且不再支持几十元一年的套餐了。因此打算寻找其它方案来替代`Bitcron`。对于新方案的要求主要有几下几个：

1. 支持将原有博客进行迁移
2. 对系统环境要求低，部署方便
3. 支持自定义域名
4. 对`Markdown`支持良好，支持数学公式

经过各种权衡，选择了`hugo + Github Actions + Github Pages`作为最终的方案。`hugo` 作为博客生成工具，`Github Actions`将`hugo`生成的博客发布到`Github Pages`。我们要做的只是在Client将新建的博客`push`到`Github`的私有仓库，剩下的工作`Github`会帮助我们自动完成。下面介绍该方案的具体实现。



### 需要准备的环境

* hugo 安装（非必需）
* Github 私有仓库（用来放Blog文件）
* Github Pages 仓库（用来存放静态页面）



### Step 1 , 生成hugo文件

假设上面提到的环境已经准备好，那么第一步是通过hugo生成一个hugo网站的文件夹。下面的命令可以生成一个`blog`的文件夹；

```shell
hugo new blog
```

如果没有安装`hugo`的话，可以在github上clone一个hugo网站的项目。



### Step 2，设置`Blog`仓库

这里假设我们的Github私有仓库的名称也是`blog`，那么首先，将本地`blog`文件夹与`blog`仓库进行关联；然后开始设置Github Actions，方式是在`blog`目录下新建文件`.github/workflows/hugo_deploy.yml`，具体内容如下：

```yaml
name: Deploy Blog

on:
  push:
    branches:
      - main

jobs:
  build: # 一项叫做build的任务

    runs-on: ubuntu-18.04 # 在最新版的Ubuntu系统下运行
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo # 安装hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.81.0'
          # extended: true

      - name: Build # 生成静态网页，生成结果在public文件夹下
        run: hugo --minify

      - name: Deploy # 发布静态网页到Github Pages仓库
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.API_TOKEN_GITHUB }}
          external_repository: username/external-repository
          publish_branch: main  # default: gh-pages
          publish_dir: ./public

```



### Step 3， 设置域名

首先在blog项目进行设置

![image-20210506155758138](2021-05-06-%E4%BD%BF%E7%94%A8GitHub%20Actions%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2hugo%E5%88%B0GitHub%20Pages.assets/image-20210506155758138-1620366363722.png)

然后在域名管理处理设置

域名的具体设置可参考[Github官方文档](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)

另外为了打开Https，需要在`blog`文件夹下创建`static/CNAME`文件内容为域名。



### 数学公式的设置

引入`mathjax`需要在页面中添加

```html
<script type="text/javascript"
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```

以及根据情况可以添加

```html
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$','$'], ['\\(','\\)']],
    displayMath: [['$$','$$'], ['\\[','\\]']],
    processEscapes: true,
    processEnvironments: true,
    skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
    TeX: { equationNumbers: { autoNumber: "AMS" },
         extensions: ["AMSmath.js", "AMSsymbols.js"] }
  }
});
</script>
```

目前大部分的markdown解析会把`\\`转义成`\`，会导致在公式中使用`\\`换行显示失效，目前我的方案是在`front-matter`中设置`markup:mmark`，`mmark`的解析会将多行`$$`转换成`\[`和`\]`，单行`$$` 转换成`\(`和`\)`配合上面的设置就可以支持完成的数据公式了。需要注意的是hugo的官方文档已经表示`mmark`将会在未来移除，不知道后续会有怎样的支持。**注意， 多行$$，前后需要换行**



### 图片使相对路径生效方式

平时我会使用`typora`来编辑`markdown`文件，在插入图片时会使用相对路径，但`hugo`上却无法显示 ，问题的原因是`hugo`默认使用`Pretty URLs`，这种模式下生成的url是这样的

```
content/posts/my_topic/post-1.md
=> example.com/posts/my_topic/post-1/
```

`post-1.md`下的相对路径会变为`example.com/my_topic/posts/post-1/_img/my_img.png`，但该图片的实际路径是`example.com/posts/my_topic/_img/my_img.png/`。

解决方案是指定url方式为`Ugly URLs`，需要做的只是在`config.toml`中设置`uglyurls = true`，这样url就会变成`example.com/my_topic/posts/post-1.html`

详细说明可参考[hugo官方文档上的介绍](https://gohugo.io/content-management/urls/#pretty-urls)



### 添加评论

这里选择的是`gitalk`，总体上按照[网上的文章](https://xbc.me/add-gittalk-to-hugo/)配置并不难。过程中遇到的最大困难是由于网页地址超过50而导致`gitalk`无法正常加载出来 ；解决方法是先转成中文，然后取前49个字。

```
id: decodeURI((location.pathname).split("/").pop()).substring(0, 49)
```

这个问题的根本原因是，`gitalk`会把`id`里面的值做为传给`github`的一个`label`参数，而`github`规定`lable`的长度是不能超过50的。一般会把`id`的值设置成网页的`uri`，但是我们的博客一般是中文的，`uri`会对中文进行编码，这样就会使`uri`变的特别长。网络上的一些解决方案是直接截断50个字符串，我的方案是转成中文后再截50个字符串，这样看起来更加优雅一些。

另外，这处修改是在git的`submodule`中进行的修改，需要进行对仓库进行`git submodule update --recursive --remote`后才会生效。（这点待确认）



### 添加归档页面

https://xbc.me/how-to-create-an-archives-page-with-hugo/

