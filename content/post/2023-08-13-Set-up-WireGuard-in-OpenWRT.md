---
author: HKL
categories:
- 默认分类
date: "2023-08-13T21:12:00+08:00"
slug: Set-up-WireGuard-in-OpenWRT
status: publish
tags:
- Networking
- Operating
title: Set up WireGuard Server in OpenWRT
---

***This article is an AI translation of the [original version](https://vnf.cc/2019/11/openwrt-wireguard-server/).***

This article is mainly to realize in the OpenWRT router system to build WireGuard server to facilitate remote connection.

Previously, we have been using Openconnect VPN in OpenWRT, because it is SSLVPN, and it is very convenient to use with CISCO's anyconnect client. Still, because now ISPs can recognize this SSL-based traffic and block the public IP, we have to think about switching to the UDP-based OpenVPN -> WireGuard VPN.

Continued: In the original article sent to the V2EX above, everyone said WireGuard's performance is better and then looked at the information. If the number of Peers is not a lot of words, realizing Server/Client type of Dial-Up VPN is still feasible, so this side of the tutorials also writes more conveniently for you!

The following two main steps:

(1) Install and configure WireGuard in OpenWRT.

(2) Configure the multi-peers program

So now it starts, the current system is using the latest OpenWRT 19.07.0-rc1, which should also be applicable to OpenWrt 18.06.4

<!--more-->

First, let's post a topology with 2 Peers connected.

![Topo][1]

1. Install and configure WireGuard in OpenWRT.

First, install all the required packages.

```bash
opkg update
opkg install wireguard luci-proto-wireguard luci-app-wireguard
```

Preset WireGuard parameters and network segments

```bash
WG_IF="wg0"
WG_PORT="51820"
WG_ADDR="192.168.9.1/24"
```

The network segment defined by `WG_ADDR` should not conflict with the existing segments on the intranet.

Configure the firewall to open the appropriate ports.

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

Generate server and client certificates.


Generate the client wgclient.pub with the Windows client and transfer it to the router.


![WireGuard Windows Client][2]

```bash
# Rename the pubkey generated by the Windows client as wgclient.pub.
echo KWb2OFp1oc/mhU6Ypzg1OFI8R0Qc/pfCdoLnGMmLdX0= > wgclient.pub
# Generate and exchange the keys
umask u=rw,g=,o=
wg genkey | tee wgserver.key | wg pubkey > wgserver.pub
wg genpsk > wg.psk
 
WG_KEY="$(cat wgserver.key)"
WG_PSK="$(cat wg.psk)"
WG_PUB="$(cat wgclient.pub)"
```

Configure the OpenWRT server network.

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

In this way, the OpenWRT configuration is complete. Next, modify the configuration of the Windows client.

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

Then the normal single Peer will have been passed.

![Peer1 Status][6]

2. Configure multi-peers program

Because Dial-Up is convenient for connecting to home, so there is no need to start more than one network segment, multiple Peers with a network segment are the most suitable. Luci can do the following configuration.

First, modify the Allow-IP setting of the Peers on OpenWRT according to the IP address of the first Peer.

![Peer1 Modify][3]

For example, if I set `Address = 192.168.9.2/24` on the client side, then change the corresponding Peer Allowed IPs on OpenWRT to 192.168.9.2/32.


Then add a new Peer, first another terminal WireGuard client to generate a set of keys, and can be configured entirely in advance!

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

Then the generated pubkey through the Web Luci configuration to the OpenWRT above on the line.

![Peer2 OpenwRT][5]

This basically completes the two-node WireGuard VPN configuration, if you need more nodes, repeat the second step on it.


refer:

1.[WireGuard basic](https://openwrt.org/docs/guide-user/services/vpn/wireguard/basic)


  [1]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/D5ujQt2.png
  [2]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/DWEsmPg.png
  [3]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/8ePVVAd.png
  [4]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/GRGpl6C.png
  [5]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/RqPezCM.png
  [6]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/PFV8Vef.png