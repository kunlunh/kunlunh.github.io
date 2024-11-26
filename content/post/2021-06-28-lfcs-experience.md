---
author: HKL
categories:
- 默认分类
date: "2021-06-28T18:15:00+08:00"
slug: lfcs-experience
status: publish
tags:
- Linux
title: 2021-06 LFCS考试经验
---

本人在2021年06月参加LFCS考试，顺利通过，一看网上比较少人讨论这考试，所以大概写写自己的考试经验。

<!--more-->

考试形式和去年参加的[CKA考试](https://www.vnf.cc/2020/06/cka-experience/) 差不多，也是根据题目直接执行相关的指令得到结果。

另外由于出入境证件都上交给了单位，这次尝试使用驾驶证+信用卡的方式也能够顺利参考。

题目是24道，66%分可以通过。

考试大纲可以参加: [https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/)

我这次考察的主要是用户管理，文件操作，分区管理，网络管理，其中分值较大且比较容易的是网络管理、Docker管理、virsh管理、硬盘管理，基本会启用、会查man手册就能做过去。

分值低但是比较复杂的是一些文件比较，文件类型，还需要知道ls的各个参数用法，crontab的准确用法，用户管理的准确用法，启用软RAID的命令。

另外要过滤结果，基本上熟练通过grep、awk就能够做到，同时有些题目可能用命令不好实现也可以通过自己判断，将结果人工敲在答案位置即可。

![LFCS Certificate][1]


  [1]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2021/06/705110ac0982.png