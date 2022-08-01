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
* Create a new page
````bash
$ hexo new [layout] <title>
````
通过`hexo new`命令可以创建一个新的page，通过可选参数`layout`可选控制页面的布局

* Generate static page and deploy
````bash
$ hexo clean && hexo g -d
````
通过此命令可以将写完的page部署到远程服务器