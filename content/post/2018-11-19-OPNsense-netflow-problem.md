---
author: HKL
categories:
- 默认分类
cid: 65
date: "2018-11-19T09:46:00Z"
slug: OPNsense-netflow-problem
status: publish
tags:
- Operating
- FreeBSD
title: OPNsense 内置Netflow工作不正常或者获取不到数据
updated: 2019/01/29 16:22:13
---


flowd_aggregate服务没有工作，或者报表没有数据 
此情况一般为日志缓存有问题，需要手动清缓存
命令如下 

```bash
rm /var/log/flowd.log*
rm /var/netflow/*.sqlite
```