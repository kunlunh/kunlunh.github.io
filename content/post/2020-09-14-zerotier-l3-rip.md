---
author: HKL
categories:
- 默认分类
date: "2020-09-14T23:32:00Z"
slug: zerotier-l3-rip
status: publish
tags:
- Linux
- Operating
- Networking
title: 用动态路由打通各Virtual L2网络
---

本文主要以Zerotier组好的各Virtual L2网络节点 + 动态路由 RIP 为例，最终效果就是让网关建立Zerotier的Virtual L2网络，网关下的其它网段就能相互通信。

拓扑如下：

![Zerotier RIP 拓扑][1]

例如如上拓扑，Zerotier建立了192.168.193.0/24的互联虚拟二层，有192.168.193.6、192.168.193.11、192.168.193.21的网关下面有172.16.0.0/23、192.168.2.0/24、192.168.11.0/24三个网段，那么通过建立rip动态路由，让这三个网段可以互通。

网关设备默认是OpenWRT设备，前提是已经通过例如VPN/Tinc/Zerotier等方法建立了互联段。

如果还没建立的话可以参考：
[/2020/03/openwrt-tinc/](/2020/03/openwrt-tinc/)

[/2019/12/zerotier-sd-lan/](/2019/12/zerotier-sd-lan/)

先建好互联的段，

下面开始做动态路由的配置

<!--more-->

先安装quagga-zebra和quagga-ripd组件：

```bash
# opkg install quagga-zebra quagga-ripd
```

如果想要有操作终端界面可以安装`quagga-vtysh`

分别编辑rip路由
`/etc/quagga/ripd.conf`

`192.168.193.6、172.16.0.0/23`
```bash
password zebra
!
router rip
 network 192.168.193.0/24
 route 172.16.0.0/23
!
access-list vty permit 127.0.0.0/8
access-list vty deny any
!
line vty
 access-class vty
```

`192.168.193.11、192.168.2.0/24`
```bash
password zebra
!
router rip
 network 192.168.193.0/24
 route 192.168.2.0/24
!
access-list vty permit 127.0.0.0/8
access-list vty deny any
!
line vty
 access-class vty
```

`192.168.193.21、192.168.11.0/24`
```bash
password zebra
!
router rip
 network 192.168.193.0/24
 route 192.168.11.0/24
!
access-list vty permit 127.0.0.0/8
access-list vty deny any
!
line vty
 access-class vty
```

然后重启quagga进程
`/etc/init.d/quagga restart`

在有装vtysh的设备可以进终端看看rip状态
```bash
# vtysh

Hello, this is Quagga (version 1.1.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

nRouter# show ip rip status
Routing Protocol is "rip"
  Sending updates every 30 seconds with +/-50%, next due in 28 seconds
  Timeout after 180 seconds, garbage collect after 120 seconds
  Outgoing update filter list for all interface is not set
  Incoming update filter list for all interface is not set
  Default redistribution metric is 1
  Redistributing:
  Default version control: send version 2, receive any version
    Interface        Send  Recv   Key-chain
    ztuku6smag       2     1 2
  Routing for Networks:
    192.168.193.0/24
  Routing Information Sources:
    Gateway          BadPackets BadRoutes  Distance Last Update
    192.168.193.21           0         0       120   00:00:19
    192.168.193.11           0         0       120   00:00:07
    192.168.193.14           0         0       120   00:00:10
  Distance: (default is 120)
nRouter#

```

在其它设备可以看看路由
```bash
# ip route | grep zebra
172.16.0.0/23 via 192.168.193.6 dev ztuku6smag proto zebra metric 20
192.168.11.0/24 via 192.168.193.21 dev ztuku6smag proto zebra metric 20
192.168.12.0/24 via 192.168.193.14 dev ztuku6smag proto zebra metric 20

```
这样子这三个网段就能够互通了，

比如从172.16.1.99 可以通 192.168.11.8
```bash 
root@ubuntu-lxc:/etc/apt# ip addr | grep inet
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
    inet 172.16.1.99/23 brd 172.16.1.255 scope global eth0
    inet6 fe80::38a9:fcff:fe1d:b7f6/64 scope link 
root@ubuntu-lxc:/etc/apt# ping 192.168.11.8
PING 192.168.11.8 (192.168.11.8) 56(84) bytes of data.
64 bytes from 192.168.11.8: icmp_seq=1 ttl=62 time=19.0 ms
64 bytes from 192.168.11.8: icmp_seq=2 ttl=62 time=14.6 ms
64 bytes from 192.168.11.8: icmp_seq=3 ttl=62 time=15.0 ms
64 bytes from 192.168.11.8: icmp_seq=4 ttl=62 time=16.1 ms
64 bytes from 192.168.11.8: icmp_seq=5 ttl=62 time=16.9 ms
^C
--- 192.168.11.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 14.592/16.344/19.022/1.572 ms
root@ubuntu-lxc:/etc/apt# 
```

  [1]: https://imgur.com/download/MDg1hzo/



