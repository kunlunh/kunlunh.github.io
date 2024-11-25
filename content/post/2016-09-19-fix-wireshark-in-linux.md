---
author: HKL
categories:
- 默认分类
cid: 40
date: "2016-09-19T10:14:00Z"
slug: fix-wireshark-in-linux
status: publish
tags:
- Operating
- Linux
title: Archlinux下解决wireshark普通用户抓包权限问题
updated: 2019/01/29 16:31:43
---


wireshark软件是一个抓包利器，但是其调用的dumpcap组件需要root权限才能使用，以普通用户打开wireshark会提示权限不足的问题。

当然可以通过`sudo wireshark`命令强行使用wireshark，在最新版会提示一下报错信息。

> Lua: Error during loading:  [string
> "/usr/share/wireshark/init.lua"]:44: dofile has been disabled due to
> running Wireshark as superuser. See
> https://wiki.wireshark.org/CaptureSetup/CapturePrivileges for help in
> running Wireshark as an unprivileged user.

所以我们可以通过新建可以用root权限使用dumpcap的用户组来解决wireshark的权限问题。

具体实现方法：

1.添加wireshark用户组

`sudo groupadd wireshark`

2.将dumpcap更改为wireshark用户组

`chgrp wireshark /usr/sbin/dumpcap`

3、让wireshark用户组有root权限使用dumpcap 

`chmod o-rx /usr/sbin/dumpcap`

4、将需要使用的普通用户名加入wireshark用户组，我的用户是“example”（需要根据具体用户名修改！），则需要使用命令：

`sudo gpasswd -a example wireshark`