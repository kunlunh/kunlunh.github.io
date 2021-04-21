---
author: HKL
categories:
- 默认分类
cid: 59
date: "2018-11-30T07:00:00Z"
slug: jcom-to-db9-female
status: publish
tags:
- Operating
- Hardware
title: MotherBoard JCOM to DB9 female
updated: 2019/02/12 14:01:03
---


Since there is a project to deploy an x86 VyOS as the router, After I install the system and I find out that I can use the console to connect to the system just like other Cisco(H3C) routers rather than using the screen.

However, The Cisco router are using RS232-RJ45 port, and I just spot that the H61 MotherBoard ais using jcom jumper to the DB9 male port. And my USB-RS232 converter is also with the DB9 male port, so I'd like to make a `jcom to DB9 female` plan.

I just find out some jcom and DB9 information here,

<!--more-->

![jcom to DB9 male img](https://img.jnuer.com/img/2018/11/image001.gif "jcom to DB9 male")

**JCOM jumper on the motherboard**

② RXD | ④ DTR | ⑥ DSR |⑧ CTS | EMPTY
------|-------|-------|------|-------
① DCD | ③ TXD | ⑤ GND |⑦ RTS |⑨ RI

![jcom jumper](https://img.jnuer.com/img/2018/11/image002.png "jcom jumper")

**JCOM line**

First is red ->  ->  -> 

①DCD ②RXD ③TXD ④DTR ⑤GND ⑥DSR ⑦RTS ⑧CTS ⑨RI

![jcom line](https://img.jnuer.com/img/2018/11/image003.gif "jcom line")

**As usual, when communicating via the RS232, a PC is as the `DTE`, but as I am using the PC being the router, it is becoming the `DCE`, so it's essential to switch the RXD and TXD.**

**Front view**

![DB9 female Front view](https://img.jnuer.com/img/2018/11/image004.png "DB9 female Front view")

**Back view**

![DB9 Back view abstract](https://img.jnuer.com/img/2018/11/20181130111135.png "DB9 Back view abstract")


![DB9 Back view](https://img.jnuer.com/img/2018/11/20181130111349.jpg "DB9 Back view")


Finally, it works well with the Vyos Serial Console Mode.

**VyOS Serial Console GRUB**

![VyOS Serial Console GRUB](https://img.jnuer.com/img/2018/11/20181130093018.png "VyOS Serial Console GRUB")

**VyOS Serial Console Systemctl**

![VyOS Serial Console Systemctl](https://img.jnuer.com/img/2018/11/20181130093039.png "VyOS Serial Console Systemctl")

**VyOS Serial Console System**

![VyOS Serial Console System](https://img.jnuer.com/img/2018/11/20181130093058.png "VyOS Serial Console System")

**The VyOS Router**

![1f63a](https://cdn.staticfile.org/twemoji/11.2.0/2/72x72/1f63a.png)

![The VyOS Router](https://img.jnuer.com/img/2018/11/20181130111944.jpg "The VyOS Router")

**Ref:**

https://wiki.vyos.net/wiki/Serial_console

https://help.ubuntu.com/community/SerialConsoleHowto

http://www.frontx.com/pro/cpx102_2.html
