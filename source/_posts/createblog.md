---
title: Hexo + github 搭建自己的博客
date: 2016-04-09 23:11:09
tags: 技术
categories: 技术
page: 'qwe'
description: 技术
layout: single-column
toc: true
---
## 1. 配置环境
 安装node，在[node官网](http://nodejs.cn/)上下载node，进行安装

 安装Git, 下载[Git](https://git-scm.com/download/)进行安装

 ## 2. 安装HEXO
 安装hexo
 ``` bash
npm install  -g hexo -cli
 ```
 进入你的blog目录，初始化配置
 ```bash
hexo init <folder>
cd blog
npm install
 ```
启动本地服务, 再浏览器查看[http://localhost:4000/](http://localhost:4000/)
```bash
hexo server
```
## 3. 部署到github上
生成静态资源
```bash
hexo generate
```
生成的静态资源在public目录下，把public的全部资源拷贝出来，push到gh-pages分支上

## 4. 参考资料：
1.[hexo官网](https://hexo.io/zh-cn/docs/)
2.[concise主题](https://github.com/HmyBmny/hexo-theme-concise)
3.[HEXO+Github,搭建属于自己的博客](http://www.jianshu.com/p/465830080ea9)
