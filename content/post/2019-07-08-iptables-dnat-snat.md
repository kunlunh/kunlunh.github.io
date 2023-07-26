---
author: HKL
categories:
- 默认分类
cid: 73
date: "2019-07-08T11:15:00Z"
slug: iptables-dnat-snat
status: publish
tags:
- Network
- Operating
title: iptables上入站流量同时启用DNAT和SNAT
updated: 2019/07/19 18:11:07
---


情况
目的主机网关是否为同一个openwrt,如果不是的话有可能是因为dnat没有对请求源地址做转换导致来回路径不一样，需要用iptables同时对dnat的到内网流量做源地址转换

```bash
iptables -t nat -A PREROUTING -d 公网IP -p tcp --dport 公网端口 -j DNAT --to-destination 内网IP:内网端口
iptables -t nat -A POSTROUTING -s 0.0.0.0/0 -o br-lan(内网网卡名字) -j SNAT --to 内网网卡接口IP
```

<!--more-->


![IMG1][1]

  [1]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/07/awh7bavqyp.png