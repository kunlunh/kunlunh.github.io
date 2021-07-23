---
author: HKL
categories:
- 默认分类
date: "2021-07-23T18:51:00+08:00"
slug: n2n-rip-virtualnet
status: publish
tags:
- Linux
- Operating
- Networking
title: n2n动态路由器异地组网方案
---

本文主要介绍通过n2n结合动态路由RIP的异地组网方案，经过近一年的使用，比较稳定，所以分享一下。


<!--more-->


###（0）前言及网络拓扑

首先简单说一下组网的拓扑: ![Topo][1]

此前在v站我的博客也有陆续发过一些异地组网的方法：

[通过 N2N 组网并运行 OSPF 动态路由 on OpenWRT](https://v2ex.com/t/734161)

[用动态路由打通各 Virtual L2（Zerotier）网络](https://v2ex.com/t/707060)

[OpenWRT 结合 tinc 组自己的 SDLAN（Step by Step）](https://v2ex.com/t/649829)

[OpenWRT 搭建 WireGuard 服务器](https://v2ex.com/t/624344)

大家收藏点赞挺多的，就是没啥回复😆

现在分享一下已经稳定运行一年多的方案，n2n + quagga-rip，方案只需一个带公网IP的服务器作握手/中继（也可以用n2n官网提供的[不推荐]），在网络环境较好的情况下基本握手后可以实现直接穿透。


###（1）安装配置n2n

[n2n软件](https://github.com/ntop/n2n) 主要实现peer-to-peer虚拟组网功能，编译快速，配置简单，稳定。一般同类的软件有zerotier, tinc, ... 本人基本都用过，综合考虑使用n2n, 其它同类软件实现功能一样。


** SuperNode 节点：**

n2n SuperNode 节点类似于zerotier的planet或者moons，用作握手或者中继，

本文拓扑中SuperNode节点使用Archlinux服务器，可直接pacman安装，其它发行版可通过包管理或者自编译安装，非常简单。

只需监听端口和community(自定义字符串，和后面配置一致)即可

```bash
$ supernode -h
Welcome to n2n v.2.8.0 for x86_64-unknown-linux-gnu
Built on Jan 22 2021 15:06:27
Copyright 2007-2020 - ntop.org and contributors

supernode <config file> (see supernode.conf)
or
supernode -l <local port> -c <path> [-u <uid> -g <gid>] [-t <mgmt port>] [-v] 

-l <port>	Set UDP main listen port to <port>
-c <path>	File containing the allowed communities.
-u <UID>	User ID (numeric) to use when privileges are dropped.
-g <GID>	Group ID (numeric) to use when privileges are dropped.
-t <port>	Management UDP Port (for multiple supernodes on a machine).
-v        	Increase verbosity. Can be used multiple times.
-h        	This help message.

```

** EdgeNode 节点：**

EdgeNode 节点运行在各接入网段网关上，本人主要是运行其在各个拨号的OpenWRT路由器网关上，这样更加便利地将各个网段互联：

OpenWRT包管理中没有新版本n2n，所以可以参考 [n2n 2.8 for OpenWRT](https://github.com/hiplon/openwrt-n2n-latest) 是OpenWRT交叉编译的脚本，也有打包好的ipk安装包，当然也可以用其它方法

安装完edge后，主要配置如下：（以拓扑中节点X为例）

```bash
root@XMOPWRT:~# cat /etc/n2n/edge.conf 
-d=tincn0
-c=myperfectn2n //与前面supernode配置的community(自定义字符串)一致
-a=10.193.111.14  //n2n互联段IP
-A1  //不启用加密性能更好（视乎需求）
-f
-r  # Enable packet forwarding  [启用 N2N 包转发需要] 
-E  # Accept multicast MAC addresses  [启用动态路由需要] 
-l=supernode.ntop.org:7777

```

** 启动n2n **

SuperNode

```bash
systemctl enable n2n
```

EdgeNode

```bash
/etc/init.d/edge enable
/etc/init.d/edge start
```

** 内网IP **

```bash
root@XMOPWRT:~# ip addr

5: br-lan: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether 66:09:80:0e:c9:af brd ff:ff:ff:ff:ff:ff
    inet 10.193.14.1/24 brd 10.193.14.255 scope global br-lan
       valid_lft forever preferred_lft forever


11: tincn0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1290 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 5e:36:f6:58:de:a8 brd ff:ff:ff:ff:ff:ff
    inet 10.193.111.14/24 brd 10.193.111.255 scope global tincn0
       valid_lft forever preferred_lft forever
    inet6 fe80::5c36:f6ff:fe58:dea8/64 scope link 
       valid_lft forever preferred_lft forever

```


至此，各个节点应该通过互联段可以互通。

```bash
root@XMOPWRT:~# ping 10.193.111.11
PING 10.193.111.11 (10.193.111.11): 56 data bytes
64 bytes from 10.193.111.11: seq=0 ttl=64 time=20.020 ms
^C
--- 10.193.111.11 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 20.020/20.020/20.020 ms
root@XMOPWRT:~# ping 10.193.111.10
PING 10.193.111.10 (10.193.111.10): 56 data bytes
64 bytes from 10.193.111.10: seq=0 ttl=64 time=15.340 ms
^C
--- 10.193.111.10 ping statistics ---
2 packets transmitted, 1 packets received, 50% packet loss
round-trip min/avg/max = 15.340/15.340/15.340 ms

```

###（2）安装配置quagga

主要通过quagga并通过RIP路由协议实现动态路由，

** 各EdgeNode节点安装quagga-ripd **

```bash
opkg install quagga-ripd quagga quagga-libzebra quagga-zebra quagga-watchquagga
```

安装完quagga后，主要配置如下：（以拓扑中节点X为例）:

```bash
root@XMOPWRT:~# cat /etc/quagga/ripd.conf
password zebra
!
router rip
 network 10.193.111.0/24
 route 10.193.14.0/24
!
access-list vty permit 127.0.0.0/8
access-list vty deny any
!
line vty
 access-class vty

```

** 启动quagga-ripd **

EdgeNode

```bash
/etc/init.d/quagga enable
/etc/init.d/quagga start
```

至此，各个EdgeNode节点的br-lan网段应该通过可以互通。

```bash
C:\Users\k>ipconfig

Windows IP 配置

无线局域网适配器 WLAN:

   连接特定的 DNS 后缀 . . . . . . . : lan
   IPv6 地址 . . . . . . . . . . . . : fd78:ecee:8a17:0:bcb2:17a9:71cd:8ea5
   临时 IPv6 地址. . . . . . . . . . : fd78:ecee:8a17:0:79c1:4:2b1:b58f
   本地链接 IPv6 地址. . . . . . . . : fe80::bcb2:17a9:71cd:8ea5%9
   IPv4 地址 . . . . . . . . . . . . : 10.193.14.133
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . : 10.193.14.1


C:\Users\k>ping 10.193.10.30

正在 Ping 10.193.10.30 具有 32 字节的数据:
来自 10.193.10.30 的回复: 字节=32 时间=14ms TTL=62
来自 10.193.10.30 的回复: 字节=32 时间=13ms TTL=62

10.193.10.30 的 Ping 统计信息:
    数据包: 已发送 = 2，已接收 = 2，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 13ms，最长 = 14ms，平均 = 13ms
Control-C
^C
C:\Users\k>ping 10.193.11.9

正在 Ping 10.193.11.9 具有 32 字节的数据:
来自 10.193.11.9 的回复: 字节=32 时间=18ms TTL=62

10.193.11.9 的 Ping 统计信息:
    数据包: 已发送 = 1，已接收 = 1，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 18ms，最长 = 18ms，平均 = 18ms
Control-C
^C
```


###（3）iptables配置

```bash
iptables -A input_rule -i tinc+ -j ACCEPT
iptables -A forwarding_rule -i tinc+ -j ACCEPT
iptables -A forwarding_rule -o tinc+ -j ACCEPT
iptables -A output_rule -o tinc+ -j ACCEPT
```

为了安全或者一些避免部分特定情况导致网络访问不通，可以启用SNAT(可选/建议)

```bash
iptables -t nat -A POSTROUTING -s 0.0.0.0/0 -o tincn0 -j SNAT --to 10.193.111.14
```


[1]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2021/07/topo20210723.png