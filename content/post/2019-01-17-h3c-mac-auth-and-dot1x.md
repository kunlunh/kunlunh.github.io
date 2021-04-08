---
author: HKL
categories:
- 默认分类
cid: 64
date: "2019-01-17T09:47:00Z"
slug: h3c-mac-auth-and-dot1x
status: publish
tags:
- Coding
- Network
title: H3C在端口同时配置MAC地址认证和802.1x
updated: 2019/02/12 13:59:53
---


1.H3C交换机是支持在同一端口同时配置MAC地址认证和802.1x认证的。

一开始由于交换机提示的问题以为在全局上只能启用MAC地址认证或者802.1x其中的一种，后来尝试只需全局启用
   
```bash
#
 port-security enable
#
```

就同时开启了MAC地址认证和802.1x

2.架构

(1)交换机端口设定801.1x和MAC auth的port-security方式，交换机会按 `PAP的MAC地址` -> `EAP的802.1x` 顺序给RADIUS Server发送请求报文，当其中有一项RADIUS检验通过并回复`Access-Accept`报文时，交换机准予终端接入


<!--more-->


![ARCH](https://img.ppuu.org/img/2019/01/20190101.png "ARCH")

(2)对大部分终端实行基于证书的802.1x验证方式接入

通过802.1x EAP方式认证接入终端

(3)部分终端不支持802.1x协议则通过MAC地址验证方式接入



3.The code


4.总结


(Updating...)
