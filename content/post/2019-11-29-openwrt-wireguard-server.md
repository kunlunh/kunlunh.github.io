---
author: HKL
categories:
- 默认分类
date: "2019-11-29T15:25:00Z"
slug: openwrt-wireguard-server
status: publish
tags:
- Networking
- Operating
title: OpenWRT搭建WireGuard服务器
---

本文主要实现在OpenWRT路由器系统下搭建WireGuard服务器方便远程连接，

之前一直是在OpenWRT使用Openconnect VPN,因为是SSLVPN使用起来结合CISCO的anyconnect客户端是很方便的，但是由于现在ISP连这种基于SSL的流量也有可以做识别并封公网IP，所以不得不考虑切换至基于UDP的OpenVPN了->WireGuard VPN。

续：原来的文章发到V站上面大家都说WireGuard的性能更好，然后看了一下资料，如果Peers数不是很多的话其实实现Server/Client类型的Dial Up VPN还是可行的，所以这边也写一下教程方便大家

下面主要分两步：

（1）在OpenWRT安装并配置好WireGuard

（2）配置多Peers方案

那么现在开始吧，目前系统是使用了最新的OpenWRT 19.07.0-rc1, 应该同时适用于OpenWrt 18.06.4

<!--more-->

先贴个实现2个Peers连接后的拓扑

![Topo][1]

1.在OpenWRT安装并配置好WireGuard

先安装好本次所需的全部软件

```bash
opkg update
opkg install wireguard luci-proto-wireguard luci-app-wireguard
```

预设WireGuard参数与网段
```bash
WG_IF="wg0"
WG_PORT="51820"
WG_ADDR="192.168.9.1/24"
```

`WG_ADDR`定义的网段不要和内网已有的网段冲突

配置防火墙开放相应端口

```bash
# Configure firewall
uci rename firewall.@zone[0]="lan"
uci rename firewall.@zone[1]="wan"
uci rename firewall.@forwarding[0]="lan_wan"
uci del_list firewall.lan.network="${WG_IF}"
uci add_list firewall.lan.network="${WG_IF}"
uci -q delete firewall.wg
uci set firewall.wg="rule"
uci set firewall.wg.name="Allow-WireGuard"
uci set firewall.wg.src="wan"
uci set firewall.wg.dest_port="${WG_PORT}"
uci set firewall.wg.proto="udp"
uci set firewall.wg.target="ACCEPT"
uci commit firewall
/etc/init.d/firewall restart
```

生成服务器和客户端证书

客户端的wgclient.pub就先用Windows的客户端生成一个，并将其传到路由器上面

![WireGuard Windows Client][2]

```bash
# 将上图Windows客户端生成的pubkey命名为wgclient.pub
echo KWb2OFp1oc/mhU6Ypzg1OFI8R0Qc/pfCdoLnGMmLdX0= > wgclient.pub
# Generate and exchange the keys
umask u=rw,g=,o=
wg genkey | tee wgserver.key | wg pubkey > wgserver.pub
wg genpsk > wg.psk
 
WG_KEY="$(cat wgserver.key)"
WG_PSK="$(cat wg.psk)"
WG_PUB="$(cat wgclient.pub)"
```

配置OpenWRT服务器网络

```bash
# Configure network
uci -q delete network.${WG_IF}
uci set network.${WG_IF}="interface"
uci set network.${WG_IF}.proto="wireguard"
uci set network.${WG_IF}.private_key="${WG_KEY}"
uci set network.${WG_IF}.listen_port="${WG_PORT}"
uci add_list network.${WG_IF}.addresses="${WG_ADDR}"
 
# Add VPN peers
uci -q delete network.wgclient
uci set network.wgclient="wireguard_${WG_IF}"
uci set network.wgclient.public_key="${WG_PUB}"
uci set network.wgclient.preshared_key="${WG_PSK}"
uci add_list network.wgclient.allowed_ips="${WG_ADDR%.*}.0/${WG_ADDR#*/}"
uci commit network
/etc/init.d/network restart
```

这样的话，OpenWRT上面就已经完成配置了，接下来修改一下Windows客户端的配置

```bash
[Interface]
PrivateKey = 6CJpj1CE2kqmfhJWu9UlzvCKqfm6g9yP8xCM+ggHCU4=
Address = 192.168.9.2/24

[Peer]
PublicKey = EI0o2k+BKTPoVP6e0hbJQSgn3gerwntlsebxLXt1Q3w=
PresharedKey = Ys1gDMulGlZAfW6HVWru5hpxmcQ3BHtWcwYV/pXeW3k=
AllowedIPs = 192.168.9.0/24, 192.168.234.0/24
Endpoint = ddns.example.com:51820
```

那样正常单Peer就已经通了。

![Peer1 Status][6]

2.配置多Peers方案

因为是方便Dial Up连回家，所以不需要起多个网段了，多个Peers用一个网段是最方便的。接下来的配置都可以通过Luci去完成了。

先根据第一个Peer中使用到的IP地址修改OpenWRT上面Peers的Allow-IP设定

![Peer1 Modify][3]

比如这个我在客户端设置 `Address = 192.168.9.2/24`,那么OpenWRT上面对应的Peer Allowed IPs修改成192.168.9.2/32就可以了，

然后再新增一个Peer,那么先再另外一台终端的WireGuard客户端上面生成一组密钥，并可提前将配置完整

![Peer2 Client][4]

```bash
[Interface]
PrivateKey = yBrwJicjkYbOIFtnbhWSoHahhPLivpekcp+u1Gmf72I=
Address = 192.168.9.3/24

[Peer]
PublicKey = EI0o2k+BKTPoVP6e0hbJQSgn3gerwntlsebxLXt1Q3w=
PresharedKey = Ys1gDMulGlZAfW6HVWru5hpxmcQ3BHtWcwYV/pXeW3k=
AllowedIPs = 192.168.9.0/24, 192.168.234.0/24
Endpoint = ddns.example.com:51820
```

然后将其生成的pubkey通过Web Luci配置到OpenWRT上面去就行了

![Peer2 OpenwRT][5]

这样基本就完成了两节点的WireGuard VPN配置，如果需要更多的节点，重复第二步就可以了。


refer:

1.[WireGuard basic](https://openwrt.org/docs/guide-user/services/vpn/wireguard/basic)


  [1]: https://img.ppuu.org/img/2019/11/20191129145744.png
  [2]: https://img.ppuu.org/img/2019/11/20191129151055.png
  [3]: https://img.ppuu.org/img/2019/11/20191129151659.png
  [4]: https://img.ppuu.org/img/2019/11/20191129152237.png
  [5]: https://img.ppuu.org/img/2019/11/20191129152552.png
  [6]: https://img.ppuu.org/img/2019/11/20191129153455.png