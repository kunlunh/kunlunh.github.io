---
author: HKL
categories:
- 默认分类
date: "2023-08-24T23:40:00Z"
slug: Set-up-OpenVPN-in-OpenWRT
status: publish
tags:
- Networking
- Operating
title: Set up OpenVPN Server in OpenWRT
---

***This article is an AI translation of the [original version](https://vnf.cc/2019/11/openwrt-openvpn-server/).***

This article is mainly realized in the OpenWRT router system to build OpenVPN server to facilitate remote connection!

Before has been in OpenWRT using Openconnect VPN, because it is SSLVPN use up with CISCO anyconnect client is very convenient, but because now even this SSL-based traffic ISPs can do to identify and seal the public IP, so have to consider switching to UDP-based OpenVPN.

Here are three main steps:

(1) install and configure OpenVPN in OpenWRT

(2) Configure a multi-user program

(3) Combine Luci to display OpenVPN.

So let's get started, the system is currently using the OpenWRT 19.07.0-rc1, which also applies to OpenWrt 18.06.4.

<!--more-->

1. Install OpenVPN in OpenWRT.

First, install all the required packages

```bash
opkg update
opkg install openvpn-easy-rsa openvpn-mbedtls luci-app-openvpn
```

Configure the firewall to open the appropriate ports

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

Generate server and client certificates

```bash
# Configuration parameters
export EASYRSA_PKI="/etc/easy-rsa/pki"
export EASYRSA_REQ_CN="ovpnca"
 
# Remove and re-initialize the PKI directory
easyrsa --batch init-pki
 
# Generate DH parameters
# Long time
easyrsa --batch gen-dh
 
# Create a new CA
easyrsa --batch build-ca nopass
 
# Generate a keypair and sign locally for a server
easyrsa --batch build-server-full server nopass
 
# Generate a keypair and sign locally for a client
easyrsa --batch build-client-full client nopass
```

Generating server configuration files

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
`OVPN_POOL="192.168.8.0 255.255.255.0"` Define a pool of addresses that do not conflict with addresses already on the intranet
`push "redirect-gateway def1"` is to use OpenVPN's gateway as the default gateway, it will create a default route pointing to OpenVPN's gateway, if you just need to access your home network, you can modify this article as needed, such as`push "route 192.168.1.0 255.255.255.0 192.168.8.1"` 

run `ip addr show tun0` and `cat /var/run/openvpn.server.status` to check the status of OpenVPN
![OpenVPN Status][2]

Generate client ovpn file

```bash
# Determine whether to use DDNS or public IP to use as an OpenVPN connection, and configure the OVPN_SERV parameter, this time using the DDNS address as an example
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

Importing this ovpn into the OpenVPN client will allow you to link to the OpenVPN server.

![Connection Status][3]

So far the general OpenVPN Server configuration has been completed, the current problem is that a certificate can only be connected to a client, the next step is to configure a multi-user program.

2.Multi-user Deployment

There are two kinds of multi-user programs, one is to generate multiple certificate files, each user to use a separate certificate; the other is to use a single certificate with a user password form.

Both of these will be posted a little configuration, because even home is mainly for convenience, so it will be the user name and password of the main way.

Multi-certificate method:

Need to generate another set of user public and private keys

```bash
# Configuration parameters
export EASYRSA_PKI="/etc/easy-rsa/pki"
 
# Add one more client
easyrsa --batch build-client-full client1 nopass
```

Find `client1.crt` in `/etc/easy-rsa/pki/issued` , find `client1.key` in `/etc/easy-rsa/pki/private`

Replace the cert of `client1.crt` and the key of `client1.key` with the <cert><key> segment of the ovpn file to generate the ovpn file for the second user.

Single certificate multi-user mode:

Creating User Authentication Scripts (checkpsw.sh)

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

Configuring Execution Privileges

`chmod +x /etc/openvpn/checkpsw.sh`

Configuring user password files

`/etc/openvpn/psw-file`

```bash
user1 passwd1
user2 passwd2
```

Modify the server-side configuration file

After `/etc/openvpn/server.conf` add

```bash
script-security 3
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env
username-as-common-name
verify-client-cert none
```

Modify the client configuration file

then remove `<cert>` and `<key>`

Add the following:

`auth-user-pass`

That way you can log in with your user password.


3.OpenWRT Luci Interations

This step is to make it easier to see OpenVPN status information in the OpenWRT web interface.

Ensure that you have installed

`opkg install luci-app-openvpn`

Modify the luci configuration with the command

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

2.[Openvpn 2.4 Setting up user password authentication](http://www.89cool.com/811.html)

  [1]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2019/11/kWcLKdG.png
  [2]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2019/11/ZpAwGkI.png
  [3]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2019/11/kGqkfsA.png
  [4]: https://cdn.jsdelivr.net/gh/kunlunh/blog-photo/2019/11/LLJYn5E.png