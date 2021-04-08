---
author: HKL
categories:
- 默认分类
cid: 39
date: "2016-09-01T14:19:00Z"
slug: install-reactos-qemu
status: publish
tags:
- Operating
- Linux
title: 在Archlinux上通过qemu运行ReactOS
updated: 2019/01/29 16:32:16
---


ReactOS是一个模拟实现Windows平台运行Windows应用的免费开源系统，按照官方的说法就是

`Imagine running your favorite Windows applications and drivers in an open-source environment you can trust. That's ReactOS. Not just an Open but also a Free operating system. `

很早就认识这个系统，不过她的稳定性还有可用性当然仍然是比较差的，只能算是技术的先行，不过我们还是要对他们的发展保持乐观的态度，

而且刚好又认识了qemu这个仿真器，就不妨试试在qemu上运行一下这个系统啦。


<!--more-->


首先在官网把ReactOS的安装镜像下载下来 [下载链接](https://sourceforge.net/projects/reactos/files/ReactOS/0.4.2/ReactOS-0.4.2-iso.zip/download)

然后在Archlinux上安装好`qemu-arch-extra`包，通过`qemu-img create -f qcow2 reactos 4G`设定好一个硬盘镜像，

然后配置好qemu可用的网络环境，我是使用桥接的方式实现的，当然还有其它的解决方案，

可参考这编wiki创建bridge [Network bridge](https://wiki.archlinux.org/index.php/Network_bridge)

再执行命令`qemu-system-x86_64 -cdrom ReactOS-0.4.2.iso -hda reactos.img -net nic -net bridge,br=bridge0`就可以运行了，

安装过程比较简单，基本只用过去Enter键一直下一步就可以了，安装好之后就可以试用了。