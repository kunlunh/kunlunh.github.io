---
author: HKL
categories:
- 默认分类
date: "2019-08-07T07:15:00Z"
slug: migrate-blog-from-typecho
status: publish
tags:
- Blog
title: 再次将Blog迁移到github pages
---

再次将博客站从self host的typecho迁移回github pages上，顺便测试了几个CDN的加速效果。

第一次将blog放到github pages应该是15年的时候结合hexo，现在github pages已经能够自动deploy jekyll了，所以将blog theme更新至后将blog后就直接能用了。

<!--more-->

用github pages + jekyll的好处是无需在本地build一次静态文件，只用更新jekyll的配置已经post markdown就能直接通过github去build，
而且现在可以直接在github web上面_post新建一个markdown文件就能更新blog了，是相当的方便的。

本文就是直接在github web上新加的。

关于CDN，测试了国内的aliyun和qcloud，国外的CF和fastly。

国内CDN加速github pages效不忍直视，比直接访问还慢，可以直接放弃。

不要看国内CDN ping值低，到了CDN请求upstream才是漫长的过程

CF加速效果非常好，ping值150-170看起来高,但是打开网站效果好，fastly免费版好像不能自定义ssl证书，只能用fastly的二级域名，那就没有必要了。

现在继续更新riseup的invite code，有需要可以找找。
