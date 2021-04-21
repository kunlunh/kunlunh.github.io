---
author: HKL
categories:
- 默认分类
date: "2019-11-28T23:21:00Z"
slug: openwrt-openvpn-server
status: publish
tags:
- Networking
- Operating
title: OpenWRT搭建OpenVPN服务器
---

本文主要实现在OpenWRT路由器系统下搭建OpenVPN服务器方便远程连接

之前一直是在OpenWRT使用Openconnect VPN,因为是SSLVPN使用起来结合CISCO的anyconnect客户端是很方便的，但是由于现在ISP连这种基于SSL的流量也有可以做识别并封公网IP，所以不得不考虑切换至基于UDP的OpenVPN了。

下面主要分三步：

（1）在OpenWRT安装并配置好OpenVPN

（2）配置多用户方案

（3）结合Luci去显示OpenVPN

那么现在开始吧，目前系统是使用了最新的OpenWRT 19.07.0-rc1, 同时适用于OpenWrt 18.06.4

<!--more-->

1.在OpenWRT安装并配置好OpenVPN

先安装好本次所需的全部软件

```bash
opkg update
opkg install openvpn-easy-rsa openvpn-mbedtls luci-app-openvpn
```

配置防火墙开放相应端口

```bash
# Configure firewall
uci rename firewall.@zone[0]="lan"
uci rename firewall.@zone[1]="wan"
uci rename firewall.@forwarding[0]="lan_wan"
uci del_list firewall.lan.device="tun0"
uci add_list firewall.lan.device="tun0"
uci -q delete firewall.vpn
uci set firewall.ovpn="rule"
uci set firewall.ovpn.name="Allow-OpenVPN"
uci set firewall.ovpn.src="wan"
uci set firewall.ovpn.dest_port="1194"
uci set firewall.ovpn.proto="udp"
uci set firewall.ovpn.target="ACCEPT"
uci commit firewall
/etc/init.d/firewall restart
```

![OpenWRT Firewall][1]

生成服务器和客户端证书

```bash
# Configuration parameters
export EASYRSA_PKI="/etc/easy-rsa/pki"
export EASYRSA_REQ_CN="ovpnca"
 
# Remove and re-initialize the PKI directory
easyrsa --batch init-pki
 
# Generate DH parameters
# 此步会较久
easyrsa --batch gen-dh
 
# Create a new CA
easyrsa --batch build-ca nopass
 
# Generate a keypair and sign locally for a server
easyrsa --batch build-server-full server nopass
 
# Generate a keypair and sign locally for a client
easyrsa --batch build-client-full client nopass
```

生成服务器配置文件

```bash
# Generate TLS PSK
OVPN_PKI="/etc/easy-rsa/pki"
openvpn --genkey --secret ${OVPN_PKI}/tc.pem
 
# Configuration parameters
OVPN_DIR="/etc/openvpn"
OVPN_PKI="/etc/easy-rsa/pki"
OVPN_DEV="$(uci get firewall.lan.device | sed -e "s/^.*\s//")"
OVPN_PORT="$(uci get firewall.ovpn.dest_port)"
OVPN_PROTO="$(uci get firewall.ovpn.proto)"
OVPN_POOL="192.168.8.0 255.255.255.0"
OVPN_DNS="${OVPN_POOL%.* *}.1"
OVPN_DOMAIN="$(uci get dhcp.@dnsmasq[0].domain)"
OVPN_DH="$(cat ${OVPN_PKI}/dh.pem)"
OVPN_TC="$(sed -e "/^#/d;/^\w/N;s/\n//" ${OVPN_PKI}/tc.pem)"
OVPN_CA="$(openssl x509 -in ${OVPN_PKI}/ca.crt)"
NL=$'\n'
 
# Configure VPN server
umask u=rw,g=,o=
grep -l -r -e "TLS Web Server Auth" "${OVPN_PKI}/issued" \
| sed -e "s/^.*\///;s/\.\w*$//" \
| while read -r OVPN_ID
do
OVPN_CERT="$(openssl x509 -in ${OVPN_PKI}/issued/${OVPN_ID}.crt)"
OVPN_KEY="$(cat ${OVPN_PKI}/private/${OVPN_ID}.key)"
cat << EOF > ${OVPN_DIR}/${OVPN_ID}.conf
verb 3
user nobody
group nogroup
dev ${OVPN_DEV}
port ${OVPN_PORT}
proto ${OVPN_PROTO}
server ${OVPN_POOL}
topology subnet
client-to-client
keepalive 10 120
persist-tun
persist-key
push "dhcp-option DNS ${OVPN_DNS}"
push "dhcp-option DOMAIN ${OVPN_DOMAIN}"
push "redirect-gateway def1"
push "persist-tun"
push "persist-key"
<dh>${NL}${OVPN_DH}${NL}</dh>
<tls-crypt>${NL}${OVPN_TC}${NL}</tls-crypt>
<ca>${NL}${OVPN_CA}${NL}</ca>
<cert>${NL}${OVPN_CERT}${NL}</cert>
<key>${NL}${OVPN_KEY}${NL}</key>
EOF
done
/etc/init.d/openvpn restart
```
`OVPN_POOL="192.168.8.0 255.255.255.0"` 定义的地址池不要和内网已有的地址冲突
`push "redirect-gateway def1"` 是将OpenVPN的网关作为默认网关，会创建默认路由指向OpenVPN的网关，如果只是需要访问家里的网络，可将这条按需要修改，如`push "route 192.168.1.0 255.255.255.0 192.168.8.1"` 

用`ip addr show tun0`和`cat /var/run/openvpn.server.status`确认一下OpenVPN运行状态

![OpenVPN Status][2]

生成客户端ovpn文件

```bash
# 先确定使用DDNS还是公网IP作为OpenVPN连接使用，并配置好OVPN_SERV参数，本次以DDNS地址为例子
OVPN_SERV="ddns.example.com"

# Configuration parameters
OVPN_DIR="/etc/openvpn"
OVPN_PKI="/etc/easy-rsa/pki"
OVPN_DEV="$(uci get firewall.lan.device | sed -e "s/^.*\s//")"
OVPN_PORT="$(uci get firewall.ovpn.dest_port)"
OVPN_PROTO="$(uci get firewall.ovpn.proto)"
OVPN_TC="$(sed -e "/^#/d;/^\w/N;s/\n//" ${OVPN_PKI}/tc.pem)"
OVPN_CA="$(openssl x509 -in ${OVPN_PKI}/ca.crt)"
NL=$'\n'
 
# Generate VPN client profiles
umask u=rw,g=,o=
grep -l -r -e "TLS Web Client Auth" "${OVPN_PKI}/issued" \
| sed -e "s/^.*\///;s/\.\w*$//" \
| while read -r OVPN_ID
do
OVPN_CERT="$(openssl x509 -in ${OVPN_PKI}/issued/${OVPN_ID}.crt)"
OVPN_KEY="$(cat ${OVPN_PKI}/private/${OVPN_ID}.key)"
cat << EOF > ${OVPN_DIR}/${OVPN_ID}.ovpn
verb 3
dev ${OVPN_DEV%%[0-9]*}
nobind
client
remote ${OVPN_SERV} ${OVPN_PORT} ${OVPN_PROTO}
auth-nocache
remote-cert-tls server
<tls-crypt>${NL}${OVPN_TC}${NL}</tls-crypt>
<ca>${NL}${OVPN_CA}${NL}</ca>
<cert>${NL}${OVPN_CERT}${NL}</cert>
<key>${NL}${OVPN_KEY}${NL}</key>
EOF
done
ls ${OVPN_DIR}/*.ovpn
```

将该ovpn导入到OpenVPN的客户端就可以链接上OpenVPN服务器

![Connection Status][3]

至此一般的OpenVPN Server配置已经完成，目前存在的问题就是一个证书只能连接上一个客户端，下一步就是将会配置多用户的方案。

2.多用户模式

多用户方案有两种，一种是生成多个证书文件，每个用户单独使用一个证书；另外一种就是使用单证书配合用户密码的形式。

这两种都会贴一下配置，因为连回家主要是为了方便，所以会以用户名密码的方式为主。

多证书方式：

需要生成另外一组用户公钥和私钥

```bash
# Configuration parameters
export EASYRSA_PKI="/etc/easy-rsa/pki"
 
# Add one more client
easyrsa --batch build-client-full client1 nopass
```

然后在`/etc/easy-rsa/pki/issued`找到`client1.crt`,在`/etc/easy-rsa/pki/private`找到`client1.key`

将`client1.crt`的cert和`client1.key`的key替换ovpn文件中的<cert><key>段即可生成给第二位用户的ovpn文件

单证书多用户模式：

创建用户认证脚本(checkpsw.sh)

`/etc/openvpn/checkpsw.sh`

```bash
#!/bin/sh
###########################################################
# checkpsw.sh (C) 2004 Mathias Sundman <mathias@openvpn.se>
#
# This script will authenticate OpenVPN users against
# a plain text file. The passfile should simply contain
# one row per user with the username first followed by
# one or more space(s) or tab(s) and then the password.

PASSFILE="/etc/openvpn/psw-file"
LOG_FILE="/etc/openvpn/openvpn-password.log"
TIME_STAMP=`date "+%Y-%m-%d %T"`

###########################################################

if [ ! -r "${PASSFILE}" ]; then
  echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}
  exit 1
fi

CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE}`

if [ "${CORRECT_PASSWORD}" = "" ]; then 
  echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
  exit 1
fi

if [ "${password}" = "${CORRECT_PASSWORD}" ]; then 
  echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
  exit 0
fi

echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1
```

配置执行权限

`chmod +x /etc/openvpn/checkpsw.sh`

配置用户密码文件

`/etc/openvpn/psw-file`

```bash
user1 passwd1
user2 passwd2
```

修改服务端配置文件

在`/etc/openvpn/server.conf`后面添加

```bash
script-security 3
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env
username-as-common-name
verify-client-cert none
```

修改客户端配置文件

删除掉`<cert>`和`<key>`

添加如下内容:

`auth-user-pass`

那样就可以使用用户密码登录了。


3.OpenWRT Luci集成

这一步主要是方便在OpenWRT的Web界面方便看到OpenVPN的状态信息

确保已经安装好

`opkg install luci-app-openvpn`

通过命令修改luci配置

```bash
# Provide VPN instance management
ls /etc/openvpn/*.conf \
| while read -r OVPN_CONF
do
OVPN_ID="$(basename ${OVPN_CONF%.*} | sed -e "s/\W/_/g")"
uci -q delete openvpn.${OVPN_ID}
uci set openvpn.${OVPN_ID}="openvpn"
uci set openvpn.${OVPN_ID}.enabled="1"
uci set openvpn.${OVPN_ID}.config="${OVPN_CONF}"
done
uci commit openvpn
/etc/init.d/openvpn restart
```

![Luci Status][4]


refer:

1.[OpenVPN basic](https://openwrt.org/docs/guide-user/services/vpn/openvpn/basic)

2.[Openvpn 2.4 设置用户密码认证](http://www.89cool.com/811.html)

  [1]: https://img.jnuer.com/img/2019/11/20191129084037.png
  [2]: https://img.jnuer.com/img/2019/11/20191129084740.png
  [3]: https://img.jnuer.com/img/2019/11/20191129090929.png
  [4]: https://img.jnuer.com/img/2019/11/20191129095816.png