---
author: HKL
categories:
- 默认分类
date: "2020-02-01T10:25:00Z"
slug: openwrt-zerotier-nat-via-iptables
status: publish
tags:
- Networking
- Operating
- Coding
title: Zerotier网卡NAT via iptables
---

Zerotier网卡NAT

包括SNAT+DNAT，以Openwrt自带iptables为例

情况举例：

目前有10节点的Zerotier终端，部分终端也是其它子网的网关，那么通过这个网关的端口可以映射子网内其它主机的端口

例如

作为网关br-lan接口地址为192.168.22.1

有一Zerotier子网地址192.168.193.6

那么如果要做一个Zerotier的子网端口映射 192.168.193.6:8080 -> 192.168.22.7:80

可以通过以下iptables规则

```bash
iptables -A input_rule -i zt+ -j ACCEPT
iptables -A forwarding_rule -i zt+ -j ACCEPT
iptables -A forwarding_rule -o zt+ -j ACCEPT
iptables -A output_rule -o zt+ -j ACCEPT

iptables -t nat -A POSTROUTING -s 0.0.0.0/0 -o br-lan -j SNAT --to 192.168.22.1

iptables -t nat -A PREROUTING -i ztuku6smag -p tcp -d 192.168.193.6 --dport 8080 -j DNAT --to-destination 192.168.22.7:80

```

希望大家身体健康


