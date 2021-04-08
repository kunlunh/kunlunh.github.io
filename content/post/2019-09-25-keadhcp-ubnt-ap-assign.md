---
author: HKL
categories:
- 默认分类
date: "2019-09-25T10:21:00Z"
slug: keadhcp-ubnt-ap-assign
status: publish
tags:
- Networking
- Operating
title: UniFi Register Device with keadhcp
---

本文主要实现通过Kea DHCP Server下发AC(UniFi Controller)信息给Ubnt AP,

Kea DHCP是由ISC推出的新品种开源DHCP Server，与传统ISC DHCP Server比较主要添加了ipv6的支持，不过目前可参考的资料较少，设备厂商一般不会对Kea DHCP进行配置的说明。

主要参考Ubnt官方文档中ISC的相关文档：

[Ubnt official manual for DHCP Option 43](https://help.ubnt.com/hc/en-us/articles/204909754-UniFi-Device-Adoption-Methods-for-Remote-UniFi-Controllers#7)

`Linux's ISC DHCP server: dhcpd.conf`
```bash
# ...
option space ubnt;
option ubnt.unifi-address code 1 = ip-address;

class "ubnt" {
        match if substring (option vendor-class-identifier, 0, 4) = "ubnt";
        option vendor-class-identifier "ubnt";
        vendor-option-space ubnt;
}

subnet 10.10.10.0 netmask 255.255.255.0 {
        range 10.10.10.100 10.10.10.160;
        option ubnt.unifi-address 201.10.7.31;  ### UniFi Controller IP ###
        option routers 10.10.10.2;
        option broadcast-address 10.10.10.255;
        option domain-name-servers 168.95.1.1, 8.8.8.8;
        # ...
}
```

参考通过Kea DHCP配置主要实现部分如下

<!--more-->

`kea-dhcp4.conf`
```json
"option-def": [
				{
					"name": "unifi-address",
					"code": 1,
					"space": "ubnt",
					"type": "ipv4-address",
					"encapsulate": ""
				}
			],



"subnet4": [
			{
				"subnet": "10.10.10.0/24",
				"pools": [{
					"pool": "10.10.10.20 - 10.10.10.200"
				}],
				"relay": {
					"ip-address": "10.10.10.1"
				},
				"option-data": [
					{
						"name": "routers",
						"data": "10.10.10.1"
					},
					{	"name": "unifi-address",
						"space": "ubnt",
						"code": 1, 
						"data": "201.10.7.31" //UniFi Controller IP
					}
					
				]
			}

		]
```

[Kea Administrator Reference Manual
](https://github.com/hiplon/esight2dingtalk)
