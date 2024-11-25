---
author: HKL
categories:
- 默认分类
cid: 53
date: "2018-07-18T20:09:00Z"
slug: OpenBSD-upgrade
status: publish
tags: null
title: OpenBSD的更新过程
updated: 2019/01/29 16:15:46
---


（以arch为amd64的服务器从6.2更新升级6.3为例）

更改bsd.rd

```bash
wget  https://openbsd.hk/pub/OpenBSD/6.3/amd64/bsd.rd -O /bsd.rd
reboot
```

启动引导新的bsd.rd
`boot> boot hd0a:/bsd.rd`
根据引导程序安装新的OpenBSD版本