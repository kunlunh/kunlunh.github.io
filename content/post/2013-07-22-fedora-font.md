---
author: HKL
date: "2013-07-22T22:00:00Z"
slug: fedora-font
tags:
- Linux
- Opensource
title: 解决fedora启动时显示cannot open font file true的办法
---


### To get rid of "cannot open font file true"

打开`/etc/default/grub` 文件 (`# nano /etc/default/grub`)

将`GRUB_CMDLINE_LINUX=`行中的`SYSFONT=True` 改为 `SYSFONT=latarcyrheb-sun16` ；

保存退出；

运行命令：

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

然后reboot (`# reboot`)

问题即可解决。

<!--more-->

 Just manually change True every place it is used as a font name in the following 3 files:

 `/boot/grub2/grub.cfg`

 `/etc/sysconfig/i18n`

 `/etc/default/grub`

> Typically it was latarcyrheb-sun16 on most systems.

 You could use latarcyrheb-sun32 if you want a bigger font for boot messagesand in console screens.

 Then,
 
```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```
