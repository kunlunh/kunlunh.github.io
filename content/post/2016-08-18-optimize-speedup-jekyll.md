---
author: HKL
categories:
- 默认分类
cid: 35
date: "2016-08-18T14:53:00Z"
slug: optimize-speedup-jekyll
status: publish
tags:
- Blog
- Reading
title: 对Jekyll博客的本地化优化
updated: 2019/01/29 16:34:27
---


由于博客从ghost转到了jekyll，很多东西需要进行优化配置。首先从网上找一个比较适合自己的模板，在github上找到了[wangana](https://github.com/iamnii/wangana)这款模板，觉得挺合适，然后就clone下来开始了改造。

首先是将一些不要的功能都精简掉，同时因为这个模板原来css、font、js等等的资源都是存在本地，考虑到这个主机是托管在国外一个128MB的迷你型VPS上(还要跑个nginx)，所以就将其能在国内找到CDN的资源，如jquery，bootstrap，fontawesome都从国内的公共CDN引用，然后再将主css传到七牛上，同时使用多说而不是disqus，同时找到了多说的完美https方案就用上了。

然后发现fontawesome的性能依旧不是特别理想，所以再将fontawesome换成了iconfont。所以目前博客在国内完全刷新大概需要2s，握手交换传输主页这个时间差不多是极限了。

最后再根据自己的需要增加了博客分页的功能，以后再根据需要作一下修正。

国内有个阿里云学生版主机，不过快毕业了，可能就废弃了，这个国外便宜主机就留着慢慢渣干性能。