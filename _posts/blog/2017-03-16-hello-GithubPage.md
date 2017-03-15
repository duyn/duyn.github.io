---
layout: post
title:  "Hello Github Pages!"
modified:
comments: true
share: true
categories: blog
excerpt:
tags: []
image:
  feature:
date:   2017-03-16 21:49:54
---

* Table of Contents
{:toc}

## 什么是Github Pages
Github Pages 是面向用户、组织和项目开放的公共静态页面搭建托管服 务，站点可以被免费托管在 Github 上，你可以选择使用 Github Pages 默认提供的域名 github.io 或者自定义域名来发布站点。Github Pages 支持 自动利用 Jekyll 生成站点，也同样支持纯 HTML 文档，将你的 Jekyll 站 点托管在 Github Pages 上是一个不错的选择。
到 [https://pages.github.com/](https://pages.github.com/) 上，看到可以创建的网站有两类，一类是为自己或者是自己的组织创建站点，就是新建一个仓库，仓库的名字叫做，username.github.io 或者是 orgnizationname.github.io ，注意这里的 username 和 orgnizationname 要严格替换成你自己的用户名或者组织名，大小写也要区分，不然就会有问题。然后就往仓库里面放页面内容就行了。第二类是为项目创建网站，这个其实主要步骤都是一样的，只不过稍微比创建用户或组织网站复杂一点点。这个人或组织网站的默认git分支是master，项目网站的默认git分支是gh-pages。

## Github Pages 原理
Github支持[Jekyll](http://duyn.github.io/blog/welcome-to-jekyll/)，在内部已经配置好了Jekyll，允许用户把Jekyll博客当作一个项目发布到Github，博客的所有文件通过git管理，用户提交时，Github自动会调用Jekyll，为用户把md或textile文件转换成可访问的html，所以用户也可以不在本地先转换格式，这样就可以通过写md文件的方式来写博客了。另外Github为用户的博客项目配置独立服务，使得可以以http://username.github.com方式访问，更可以设置CNAME指定另外的域名，还有很多功能，需要慢慢去发掘。可以简单点理解，Jekyll是一套博客系统，而Github提供博客托管服务。


