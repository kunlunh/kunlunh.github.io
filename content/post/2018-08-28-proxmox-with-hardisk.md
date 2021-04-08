---
author: HKL
categories:
- 默认分类
cid: 55
date: "2018-08-28T22:13:00Z"
slug: proxmox-with-hardisk
status: publish
tags:
- Operating
- Linux
- Virtualization
title: Proxmox PVE 为VM挂载物理硬盘
updated: 2019/01/29 16:23:29
---


1、查询物理硬盘标号
`ls /dev/disk/by-id/`
假设搜到的硬盘ID为”ata-ST1000DM010-ABCDE_FGHJKL”

2、挂载硬盘到虚拟机中
`qm set 100 --sata2 /dev/disk/by-id/ata-ST500DM002-1BD142_S2AKZ4ML`

其中100为虚拟机ID号