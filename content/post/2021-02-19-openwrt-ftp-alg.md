---
author: HKL
categories:
- 默认分类
date: "2021-02-19T19:01:00Z"
slug: openwrt-ftp-alg
status: publish
tags:
- Linux
- Operating
- Networking
title: 原版OpenWRT启用FTP ALG功能
---

本文主要介绍原版OpenWRT系统使用FTP ALG功能。

<!--more-->

安装对应kernel mod
```bash
root@OpenWrt:~# opkg install kmod-nf-nathelper-extra
root@OpenWrt:~# opkg install kmod-nf-ipvs-ftp
```

启用对应配置
```bash
root@OpenWrt:~# cat /etc/sysctl.d/11-nf-conntrack.conf 
net.netfilter.nf_conntrack_helper=1
```