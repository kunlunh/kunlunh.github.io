---
author: HKL
categories:
- 默认分类
date: "2020-03-04T11:12:00Z"
slug: openwrt-tinc
status: publish
tags:
- Networking
- Operating
title: OpenWRT结合tinc组自己的SDLAN
---

本文主要实现在OpenWRT路由器以及不同系统下通过tinc switch mode搭建SDLAN内网服务器方便远程连接，

Switch Mode相对来说配置比较简单，各节点均在同一广播域内，方便调控，tinc节点本身通过DNAT+SNAT可以实现对不同网间端口的调通，

同时Switch Mode中各节点的hosts文件只需保证在公网地址的节点中全部拥有维护即可，其他节点只需维护本节点以及公网节点的hosts文件

下面主要分三步：

（1）公网节点的部署(Master节点)

（2）其他节点的部署(Slave节点)

（3）节点的NAT配置

本次搭建的拓扑以下为例，两个Master节点，若干个Slave节点（以3个不同操作系统的为例）

<!--more-->

![Topo][1]



（0）tinc的安装

各大Linux发行版基本都可以通过包管理对tinc进行安装

```bash
sudo yum install tinc
sudo apt install tinc 
```



OpenWRT也可通过`opkg`安装tinc

```bash
opkg update
opkg install tinc
```



Windows可在官网下载

[https://www.tinc-vpn.org/packages/windows/tinc-1.1pre17-install.exe]: https://www.tinc-vpn.org/packages/windows/tinc-1.1pre17-install.exe	"windows tinc download link"



Windows中自带的TAP-Windwos版本比较低，建议可以考虑另外安装版本较新的TAP-Windows新建虚拟网卡而不是用tinc-vpn安装包中自带的TAP-Windows



（1）公网节点的部署(Master节点)

需要预先定义定义一个网络名 本次以`tincnet`为例`NETNAME = tincnet ` 

每个节点均需要以以下目录结构创建好配置文件夹

`/etc/tinc/tincnet`

```bash
 % ls -la
total 24
drwxr-xr-x 3 root root 4096 Mar  4 15:07 .
drwxr-xr-x 4 root root 4096 Mar  4 15:06 ..
drwxr-xr-x 2 root root 4096 Mar  4 15:06 hosts
-rwxr-xr-x 1 root root  198 Mar  4 15:06 tinc.conf
-rwxr-xr-x 1 root root   72 Mar  4 15:06 tinc-down
-rwxr-xr-x 1 root root   81 Mar  4 15:06 tinc-up

```

tinc.conf为tinc的配置文件,tinc-down,tinc-up为启动tinc时执行的脚本，一般用作启动网络，hosts文件夹中存的是各个结点的连接交换信息。



下面先说其中一个节点`Linux_Public_Node(2.2.2.2)`

各个文件配置情况：

`tinc.conf`

```bash
 % cat tinc.conf 
Name = Linux_Public_Node #此节点名称为Linux_Public_Node
AddressFamily = ipv4 #Internet走IPv4协议
BindToAddress = * 11001 #监听端口
Interface = tinctun0 #tincnet虚拟网卡
Device = /dev/net/tun 
#Mode = <router|switch|hub> (router)
Mode = switch #设置使用Swtich模式 默认为router
ConnectTo = OpenWRT_Public_Node  #连接另一公网Master节点保持双活
Cipher = aes-128-cbc #对称加密算法
```



`tinc-up` tinc启动脚本，给对应网卡加IP

```bash
 % cat tinc-up
#!/bin/sh
ip link set $INTERFACE up
ip addr add 192.168.212.8/24 dev $INTERFACE
```



`tinc-down` tinc停止脚本，关停对应网卡

```bash
 %  cat tinc-down 
#!/bin/sh
ip addr del 192.168.212.8/24 dev $INTERFACE
ip link set $INTERFACE down
```



`hosts文件夹` 主要保存各节点的交换信息，由于是第一次创建，里面应该是空文件夹，需要先创建一个自己节点的链接信息

```bash
 cd hosts
 touch Linux_Public_Node
```

```bash
 % cat Linux_Public_Node 
Address = 2.2.2.2 #公网地址
Subnet = 192.168.212.8/32 #tincnetIP信息
Port = 11001 #公网监听端口
```

创建完成后通过tincd生成非对称密钥信息

```bash
 % sudo tincd -n tincnet -K
Generating 2048 bits keys:
.............+++++ p
........................+++++ q
Done.
Please enter a file to save private RSA key to [/etc/tinc/tincnet/rsa_key.priv]: 
Please enter a file to save public RSA key to [/etc/tinc/tincnet/hosts/Linux_Public_Node]: 

```

现在`tincnet`文件夹中会生成私钥，对应的公钥信息会补全到`host/Linux_Public_Node`中

```bash
 % ls /etc/tinc/tincnet                    
hosts  rsa_key.priv  tinc.conf	tinc-down  tinc-up

 % cat /etc/tinc/tincnet/hosts/Linux_Public_Node 
Address = 2.2.2.2 
Subnet = 192.168.212.8/32
Port = 11001
-----BEGIN RSA PUBLIC KEY-----
MIIBCgKCAQEAp7F+8s8lukRv0qaE5hzrQmuy2MPb8hlte/G0pcfnBCVjIL5foJ7P
LZQrTGTsKjRbPzJ9gfZUXiZRkaA+G6Q4DBOVEt41cTceZTgAzL3ief3H6MNXQ0xW
1Wo8kDNlg6g+QJq8iV5j7adJnEPivrDm4CWl8MRmVOckisnQbseKXeuzIYDhpZLA
nlIIGMzhk3OZoPn2xpdMbJqbR0K6SrPvYq7sT3eLn0NVUbyo9D1dmtwtOJy8wmaf
oYdwTvrMdXhNNUmemnswJt8T2j8rAerqnjqz5itN8dk9mZMTKLFZ44CNnJ8jl5pE
ma8lfUnAA/Qq7i9t74pVEvWcLg8HIry16QIDAQAB
-----END RSA PUBLIC KEY-----
```

至此，节点`Linux_Public_Node(2.2.2.2)`中的配置已经完成，



下面配置另外一个节点`OpenWRT_Public_Node(1.1.1.1)`

主要的配置文件生成过程节点Linux_Public_Node类似

生成后如下：

```bash
ls -la /etc/tinc/tincnet/
drwxr-xr-x    3 root     root          4096 Mar  4 15:32 .
drwxr-xr-x    4 root     root          4096 Mar  4 15:29 ..
drwxr-xr-x    2 root     root          4096 Mar  4 15:32 hosts
-rw-------    1 root     root          1680 Mar  4 15:32 rsa_key.priv
-rwxr-xr-x    1 root     root            72 Mar  4 15:30 tinc-down
-rwxr-xr-x    1 root     root            80 Mar  4 15:30 tinc-up
-rw-r--r--    1 root     root           218 Mar  4 15:31 tinc.conf

ls -la /etc/tinc/tincnet/hosts
drwxr-xr-x    2 root     root          4096 Mar  4 15:32 .
drwxr-xr-x    3 root     root          4096 Mar  4 15:32 ..
-rw-r--r--    1 root     root           484 Mar  4 15:32 OpenWRT_Public_Node

cat /etc/tinc/tincnet/tinc.conf 
Name = OpenWRT_Public_Node
AddressFamily = ipv4
BindToAddress = * 11001
Interface = tinctun0
Device = /dev/net/tun
#Mode = <router|switch|hub> (router)
Mode = switch
ConnectTo = Linux_Public_Node
Cipher = aes-128-cbc

cat /etc/tinc/tincnet/tinc-up
#!/bin/sh
ip link set $INTERFACE up
ip addr add 192.168.212.6/24 dev $INTERFACE

cat /etc/tinc/tincnet/tinc-down
ip addr del 192.168.212.6/24 dev $INTERFACE
ip link set $INTERFACE down

cat /etc/tinc/tincnet/hosts/OpenWRT_Public_Node 
Address = 1.1.1.1
Subnet = 192.168.212.6/32
Port = 11001
-----BEGIN RSA PUBLIC KEY-----
MIIBCgKCAQEA6Tzot1eXupi+NRCfr29iKbgiXEMW1Ol327WOrAwRtiwGgQIx8LcL
iy9m+sZEWVzlfvhMub6RVM4xlZ39ghYn2OFP4x9K4D6O/HTZHbamuLOEG5zRyVGK
EN+tTStIeEaiHad04QR+6ZFB+UO7WFcBzwVh/rysOL96KaUoU9VeYHVAIkubNsvA
aNSFbmqGYpl5FrXv+sJjMyGRXjc9Lb3q/FWmPApvo/9FTElHx0xH7wvAZnc7mTCH
DB6DN62A1McgydGpn7NLnuFFEeVQf3SI9TqvajcA3vXS8P9RWuRoF5HivZIL5Ebn
FJg0UkyJcWXHUNRczdfTACF6ha0ewk8T9QIDAQAB
-----END RSA PUBLIC KEY-----
```



OpenWRT下需要再对`/etc/config/tinc`进行以下修改

```bash
cat /etc/config/tinc 
config tinc-net tincnet
	option enabled 1
	option Name OpenWRT_Public_Node

config tinc-host OpenWRT_Public_Node
	option enabled 1
	option net tincnet
```



下面要做的就是先将两个Master节点的hosts文件夹各自补充对方的节点信息，简单来说就是复制自己那份过去对面，保证两个节点的hosts文件夹都有全部节点的hosts信息

```bash
% ls -la /etc/tinc/tincnet/hosts 
total 16
drwxr-xr-x 2 root root 4096 Mar  4 15:37 .
drwxr-xr-x 3 root root 4096 Mar  4 15:25 ..
-rw-r--r-- 1 root root  486 Mar  4 15:25 Linux_Public_Node
-rw-r--r-- 1 root root  485 Mar  4 15:37 OpenWRT_Public_Node

% cat Linux_Public_Node 
Address = 2.2.2.2 
Subnet = 192.168.212.8/32
Port = 11001
-----BEGIN RSA PUBLIC KEY-----
MIIBCgKCAQEAp7F+8s8lukRv0qaE5hzrQmuy2MPb8hlte/G0pcfnBCVjIL5foJ7P
LZQrTGTsKjRbPzJ9gfZUXiZRkaA+G6Q4DBOVEt41cTceZTgAzL3ief3H6MNXQ0xW
1Wo8kDNlg6g+QJq8iV5j7adJnEPivrDm4CWl8MRmVOckisnQbseKXeuzIYDhpZLA
nlIIGMzhk3OZoPn2xpdMbJqbR0K6SrPvYq7sT3eLn0NVUbyo9D1dmtwtOJy8wmaf
oYdwTvrMdXhNNUmemnswJt8T2j8rAerqnjqz5itN8dk9mZMTKLFZ44CNnJ8jl5pE
ma8lfUnAA/Qq7i9t74pVEvWcLg8HIry16QIDAQAB

% cat OpenWRT_Public_Node 
Address = 1.1.1.1
Subnet = 192.168.212.6/32
Port = 11001
-----BEGIN RSA PUBLIC KEY-----
MIIBCgKCAQEA6Tzot1eXupi+NRCfr29iKbgiXEMW1Ol327WOrAwRtiwGgQIx8LcL
iy9m+sZEWVzlfvhMub6RVM4xlZ39ghYn2OFP4x9K4D6O/HTZHbamuLOEG5zRyVGK
EN+tTStIeEaiHad04QR+6ZFB+UO7WFcBzwVh/rysOL96KaUoU9VeYHVAIkubNsvA
aNSFbmqGYpl5FrXv+sJjMyGRXjc9Lb3q/FWmPApvo/9FTElHx0xH7wvAZnc7mTCH
DB6DN62A1McgydGpn7NLnuFFEeVQf3SI9TqvajcA3vXS8P9RWuRoF5HivZIL5Ebn
FJg0UkyJcWXHUNRczdfTACF6ha0ewk8T9QIDAQAB
-----END RSA PUBLIC KEY-----
```



最后通过systemctl，OpenWRT通过RC启动tinc, 并互ping测试一下

```bash
#Linux_Public_Node systemctl
systemctl start tinc@tincnet
#OpenWRT_Public_Node rc
/etc/init.d/tinc start
```

ping from Linux_Public_Node(192.168.212.8) to OpenWRT_Public_Node(192.168.212.6)

![Topo][3]

ping from OpenWRT_Public_Node(192.168.212.6) to Linux_Public_Node(192.168.212.8)

![Topo][2]



（2）其他节点的部署(Slave节点)

Linux系统以节点`OpenWRT_Internal_Node(192.168.212.12)`为例

同样，先按照之前的文件夹结构创建好对应目录，并复制两个Master节点hosts信息到hosts文件夹，

```bash
ls -la /etc/tinc/tincnet/
drwxr-xr-x    3 root     root             0 Mar  4 16:01 .
drwxr-xr-x    4 root     root             0 Mar  4 15:52 ..
drwxr-xr-x    2 root     root             0 Mar  4 16:01 hosts
-rw-------    1 root     root          1676 Mar  4 16:01 rsa_key.priv
-rwxr-xr-x    1 root     root            74 Mar  4 15:58 tinc-down
-rwxr-xr-x    1 root     root            82 Mar  4 15:58 tinc-up
-rw-r--r--    1 root     root           209 Mar  4 16:00 tinc.conf

ls -la /etc/tinc/tincnet/hosts/
drwxr-xr-x    2 root     root             0 Mar  4 16:01 .
drwxr-xr-x    3 root     root             0 Mar  4 16:01 ..
-rw-r--r--    1 root     root             0 Mar  4 15:58 Linux_Public_Node
-rw-r--r--    1 root     root           454 Mar  4 16:01 OpenWRT_Internal_Node
-rw-r--r--    1 root     root             0 Mar  4 15:58 OpenWRT_Public_Node

cat /etc/tinc/tincnet/
hosts/        rsa_key.priv  tinc-down     tinc-up       tinc.conf

cat /etc/tinc/tincnet/tinc.conf 
Name = OpenWRT_Internal_Node 
Interface = tinctun0
Device = /dev/net/tun
#Mode = <router|switch|hub> (router)
Mode = switch
ConnectTo = Linux_Public_Node #此处需要配置链接到两个主节点
ConnectTo = OpenWRT_Public_Node #此处需要配置链接到两个主节点
Cipher = aes-128-cbc

cat /etc/tinc/tincnet/tinc-up
#!/bin/sh
ip link set $INTERFACE up
ip addr add 192.168.212.12/24 dev $INTERFACE

cat /etc/tinc/tincnet/tinc-down
ip addr del 192.168.212.12/24 dev $INTERFACE
ip link set $INTERFACE down

cat /etc/tinc/tincnet/hosts/OpenWRT_Internal_Node 
Subnet = 192.168.212.21/32 #只需要配置Subnet参数

-----BEGIN RSA PUBLIC KEY-----
MIIBCgKCAQEAnU1maDEvbyC2XJLC8aiiwixR+einVu9gyJ4Pi1uhNMSJuVHB0HLQ
s16eOJvoEeJ4q6x0YLwjVJLlcLRW46wUAr1eMLjiovGKcYL8fZCg+Agms3+0y2SM
MaKi5fgBKjXLhdeBx4pvLaBlgYz4BP7pcVLgI0/NHBR6K1PClUtYDN1xCt5SOpiF
XIwyIawwIs6mxLknm7M0a68j7e3ovIsBOW7nLVL0GpLXVJBjAbs5z00uNOVaNJkz
tvttShGgaa+B6o1Xy8gLwB84wKNUXZbmkLobOK7h0qYgEmnQscR8Rhw5G9UJfU8G
8nrPdRRCZnDR5xRpuy0rRJG7gAzpEJ9kHwIDAQAB
-----END RSA PUBLIC KEY-----

#以下为OpenWRT系统需要配置
cat /etc/config/tinc 
config tinc-net tincnet
	option enabled 1
	option Name OpenWRT_Internal_Node

config tinc-host OpenWRT_Internal_Node
	option enabled 1
	option net tincnet
```

然后需要复制hosts文件夹的本节点信息`host\OpenWRT_Internal_Node`到Master节点的hosts文件夹中，重启tinc服务即可通，

```bash
ping 192.168.212.8
PING 192.168.212.8 (192.168.212.8): 56 data bytes
64 bytes from 192.168.212.8: seq=0 ttl=64 time=25.108 ms
64 bytes from 192.168.212.8: seq=1 ttl=64 time=8.567 ms
64 bytes from 192.168.212.8: seq=2 ttl=64 time=8.891 ms
64 bytes from 192.168.212.8: seq=3 ttl=64 time=8.745 ms
^C
--- 192.168.212.8 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 8.567/12.827/25.108 ms

ping 192.168.212.6
PING 192.168.212.6 (192.168.212.6): 56 data bytes
64 bytes from 192.168.212.6: seq=0 ttl=64 time=7.328 ms
64 bytes from 192.168.212.6: seq=1 ttl=64 time=6.871 ms
64 bytes from 192.168.212.6: seq=2 ttl=64 time=7.205 ms
64 bytes from 192.168.212.6: seq=3 ttl=64 time=7.130 ms
^C
--- 192.168.212.6 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 6.871/7.133/7.328 ms
```



再配置一个Windows系统的,

首先需要新增一个TAP-Windows的虚拟网卡，以另外安装的新版本TAP-Windows驱动为例，管理员权限运行CMD

```bash
C:\Users\k>cd C:\Program Files\TAP-Windows\bin

C:\Program Files\TAP-Windows\bin>.\addtap.bat

C:\Program Files\TAP-Windows\bin>rem Add a new TAP virtual ethernet adapter

C:\Program Files\TAP-Windows\bin>"C:\Program Files\TAP-Windows\bin\tapinstall.exe" install "C:\Program Files\TAP-Windows\driver\OemVista.inf" tap0901
Device node created. Install is complete when drivers are installed...
Updating drivers for tap0901 from C:\Program Files\TAP-Windows\driver\OemVista.inf.
Drivers installed successfully.

C:\Program Files\TAP-Windows\bin>pause
请按任意键继续. . .

```

到网络连接管理中重命名网卡名称并手动配置IP地址

![Topo][4]

![Topo][5]

然后创建好文件目录

```bash
C:\Program Files\tinc\tincnet 的目录

2020/03/04  16:14    <DIR>          .
2020/03/04  16:14    <DIR>          ..
2020/03/04  16:16    <DIR>          hosts
2020/03/04  16:17               167 tinc.conf
               1 个文件            167 字节
               3 个目录 144,868,106,240 可用字节
               
C:\Program Files\tinc\tincnet\hosts 的目录

2020/03/04  16:16    <DIR>          .
2020/03/04  16:16    <DIR>          ..
2020/03/04  16:16               499 Linux_Public_Node
2020/03/04  16:16               496 OpenWRT_Public_Node
2020/03/04  16:16                27 Windows_Internal_Node
               3 个文件          1,022 字节
               2 个目录 144,864,964,608 可用字节
```



`C:\Program Files\tinc\tincnet\tinc.conf`

```bash
Name = Windows_Internal_Node
Interface = tinctun0
#Mode = <router|switch|hub> (router)
Mode = switch
ConnectTo = OpenWRT_Public_Node
ConnectTo = Linux_Public_Node
```



`C:\Program Files\tinc\tincnet\hosts\Windows_Internal_Node`

```bash
Subnet = 192.168.212.116/32
```



生成密钥

```
C:\Program Files\tinc>.\tinc.exe -n tincnet
tinc.tincnet> generate-rsa-keys
Generating 2048 bits keys:
...................................................+++ p
......................+++ q
Done.
Please enter a file to save private RSA key to [C:/Program Files\tinc\tincnet\rsa_key.priv]:
Please enter a file to save public RSA key to [C:/Program Files\tinc\tincnet\hosts\Windows_Internal_Node]:
tinc.tincnet> quit

C:\Program Files\tinc>
```

然后将带公钥信息的Windows_Internal_Node复制到两个Master节点上面重启节点

通过Windows计算机管理中的服务启动tinc

![Topo][6]



PING其他Slave节点测试

```
C:\Program Files\tinc>ping 192.168.212.12

正在 Ping 192.168.212.12 具有 32 字节的数据:
来自 192.168.212.12 的回复: 字节=32 时间=12ms TTL=64
来自 192.168.212.12 的回复: 字节=32 时间=11ms TTL=64
来自 192.168.212.12 的回复: 字节=32 时间=12ms TTL=64
来自 192.168.212.12 的回复: 字节=32 时间=11ms TTL=64

192.168.212.12 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 11ms，最长 = 12ms，平均 = 11ms

```



如果还有新增节点，那么只需在节点本地创建好配置文件以及hosts信息，然后将本节点的hosts信息复制到Master节点上面即可。



（3）节点的NAT配置

这个是补充内容，比如Slave节点OpenWRT_Internal_Node的br-lan网卡有另一网段192.168.1.0/24的地址192.168.1.1，那么如果我想在Windows_Internal_Node通过OpenWRT_Internal_Node的 tincnet地址192.168.212.12:8080直接访问OpenWRT_Internal_Node 192.168.1.0/24网段中的192.168.1.20:80，那么可以可以通过NAT直接实现。

具体iptables配置如下：

```bash
iptables -A input_rule -i tinctun+ -j ACCEPT
iptables -A forwarding_rule -i tinctun+ -j ACCEPT
iptables -A forwarding_rule -o tinctun+ -j ACCEPT
iptables -A output_rule -o tinctun+ -j ACCEPT

iptables -t nat -A PREROUTING -i tinctun0 -p tcp -d 192.168.212.12 --dport 8080 -j DNAT --to-destination 192.168.1.20:80
iptables -t nat -A POSTROUTING -s 192.168.212.0/24 -o br-lan -j SNAT --to 192.168.1.1
```





[1]: https://img.ppuu.org/img/2020/03/20200304143813.png
[2]: https://img.ppuu.org/img/2020/03/20200304154535.png
[3]: https://img.ppuu.org/img/2020/03/20200304154619.png
[4]: https://img.ppuu.org/img/2020/03/20200304161158.png
[5]: https://img.ppuu.org/img/2020/03/20200304161220.png
[6]: https://img.ppuu.org/img/2020/03/20200304162426.png
