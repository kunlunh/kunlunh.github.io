---
author: HKL
categories:
- 默认分类
date: "2020-12-10T16:01:00Z"
slug: ospf-via-n2n-on-openwrt
status: publish
tags:
- Linux
- Operating
- Networking
title: 通过N2N组网并运行OSPF动态路由
---

本文主要以通过N2N组二层网并在其上运行OSPF动态路由，最终效果使得运行N2N的各个节点下的子网可以经路由实现互通。

Chapter 0：
这个运行N2N的节点均为OpenWRT设备，因为OpenWRT官方源已经没有N2N软件了，所以先基于N2N 2.8的源代码编译了N2N for OpenWRT, 交叉编译的Makefile以及预编译的ipk安装包可以在此下载： [Github hiplon/openwrt-n2n-latest](https://github.com/hiplon/openwrt-n2n-latest)


Chapter 1：

N2N的配置比较简单，Supernode部分以及Edge的基础部分可以参考 [Github ntop/n2n](https://github.com/ntop/n2n) ，不过由于此次需要经过N2N的虚拟网络作数据包的转发以及动态路由，所以需要启动Enable packet forwarding功能以及Accept multicast MAC addresses，具体可以参考以下配置文件：


<!--more-->

`cat /etc/n2n/edge.conf`
```bash
-d=n2ntun0
-c=myn2nnetwork
-k=mysecret
-a=10.1.0.5
-f  
-r  # Enable packet forwarding 【启用N2N包转发需要】
-E  # Accept multicast MAC addresses 【启用动态路由需要】
-l=supernode.ntop.org:7777
```

Chapter 2：

配置OSPF动态路由：

配置OSPF动态路由的拓扑可以参考我之前关于Zerotier + RIP的文章 

安装quagga-ospfd

编辑ospf路由(以其中一个节点为例)
`/etc/quagga/ospfd.conf`
```bash
password zebra
!
interface br-lan
!
interface n2ntun0
!
router ospf
 ospf router-id 10.1.0.5
 network 192.168.14.0/24 area 0.0.0.2
 network 10.1.0.0/24 area 0.0.0.0
!
access-list vty permit 127.0.0.0/8
access-list vty deny any
!
line vty
 access-class vty
```

然后重启quagga进程
`/etc/init.d/quagga restart`


