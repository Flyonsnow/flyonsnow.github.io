---
title: How to start with HEXO
date: 2022-08-01 14:03:46
tags: 
- hexo
- guide
categories:
- Guide
---

在开始正式writing之前，先记录下如何使用HEXO。

<!-- more -->


## Basic Command
###  Create a new page
````bash
$ hexo new [layout] <title>
````
通过`hexo new`命令可以创建一个新的page，通过可选参数`layout`可选控制页面的布局

| Option       | Description                              |
|--------------|------------------------------------------|
| -p,--path    | Post path,Customize the path of the post |
| -r,--replace | Replace the current post if existed      |
| -s,--slug    | Post slug.Customize the URL of the post  |

````bash
hexo new page -p about/me "About me"
````
will create `source/about/me.md`


###  Generate
````bash
$ hexo generate
````

### Server
````bash
$ hexo server
````

### Deploy
````bash
$ hexo deploy
````
可简写为`hexo d`

* Preview at localhost
````bash
$ hexo clean
$ hexo generate
$ hexo server
````
如果想更改端口`hexo server -p 5000`
或简写为
````bash
$ hexo clean && hexo g && hexo s
````
先通过`g`生成静态页面, 再通过`s`本地启动, 即可通过`http://localhost:4000`进行预览, 没有问题就可以进行部署了


### Generate static page and deploy
````bash
$ hexo clean && hexo g -d
````
通过此命令可以将写完的page部署到远程服务器


## 参考文档

1. 知乎——[从零开始搭建个人博客(超详细)](https://zhuanlan.zhihu.com/p/102592286)——枫叶
2. [hexo.io](https://hexo.io/docs/commands)