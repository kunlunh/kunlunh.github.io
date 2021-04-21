---
author: HKL
categories:
- 默认分类
date: "2019-12-26T19:25:00Z"
slug: zerotier-sd-lan
status: publish
tags:
- Networking
- Operating
- SDN
title: 通过Zerotier跨NAT组大内网
---

通过Zerotier组大内网

目前加了7个活跃节点进来，2台PC，一台腾讯云服务器，4台OpenWRT路由器。

腾讯云和其中一台有公网IP的OpenWRT作为Moons卫星中继，

测试了两个不同地方的内网设备，带宽能跑到40.8 Mbits/sec，超出预期的效果。

<!--more-->

总的来说部署模式可以视作"SD-LAN"，通过UDP转发数据，只需在握手过程通过中级，通讯不需要通过中继

节点概况

![Peers][1]
![list Peers][3]

测速情况

![Speed Test][2]

有一点值得注意的是，如果两个moons之间互加，那么存在一种情况就是如果zerotier的planet服务器出了问题之后，这两台moons有可能不会连接上planet。

不知道是zerotier的判定有bug还是怎样。




  [1]: https://img.jnuer.com/img/2019/12/20191226174610.png
  [2]: https://img.jnuer.com/img/2019/12/20191226174632.png
  [3]: https://img.jnuer.com/img/2019/12/20191226174952.png
