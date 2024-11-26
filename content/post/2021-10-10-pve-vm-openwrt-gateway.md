---
author: HKL
categories:
- 默认分类
date: "2021-10-10T10:10:00+08:00"
slug: pve-vm-openwrt-gateway
status: publish
tags:
- Linux
- Operating
- Networking
title: 一种基于虚拟机做网关的组网方案(PVE+OpenWRT VM为例)
---

本文主要介绍一种基于虚拟化平台使用VM作网关的组网方案(本文以PVE作为Host,OpenWRT作为VM Gateway为例)。


<!--more-->


**（0）前言及网络拓扑**

首先简单说一下组网的拓扑:

![Topo][4]



**（1）PVE配置**

双网卡配置如下：

![PVE config][1]


**（2）网关虚拟机配置**

![Openwrt VM config][2]


**（3）网关虚拟机状态**

![Openwrt VM status][3]


[1]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2021/10/20211011104136.png
[2]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2021/10/20211011104213.png
[3]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2021/10/20211011104240.png
[4]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2021/10/20211011110246.png