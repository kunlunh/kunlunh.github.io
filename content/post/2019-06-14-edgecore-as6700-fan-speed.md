---
author: HKL
categories:
- 默认分类
cid: 71
date: "2019-06-14T15:11:00Z"
slug: edgecore-as6700-fan-speed
status: publish
tags:
- Network
title: EdgeCore AS6700 自带系统 AOS 风扇速度修改命令
updated: 2019/06/28 17:07:05
---


1. 登入 AS6700 ( telnet [swithc_ip] )
2. 指令 linux shell
3. 使用以下指令调整转速：
cd /initrd/usr/sbin/
 
** 以下指令择一使用，最后一个值越小，风扇转速越慢
./i2cmw 1 0x35 0x20 0x07 <-- 设定为高转速(目前机台上使用的设定)
./i2cmw 1 0x35 0x20 0x06 <-- 大概就会降一半
./i2cmw 1 0x35 0x20 0x00 <-- 完全关掉(不建议)
 
** 以下指令可以查询目前的值
./i2cmd 1 0x35 0x20 <---可以查看目前的值


查看系统温度:
console# show system 