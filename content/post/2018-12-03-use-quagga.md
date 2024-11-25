---
author: HKL
categories:
- 默认分类
cid: 60
date: "2018-12-03T07:00:00Z"
slug: use-quagga
status: publish
tags: null
title: 配置使用Quagga
updated: 2019/01/29 16:01:25
---


1.add the user/group quagga
[On RedHat/CentOS]
	groupadd -g 92 quagga
	groupadd -r -g 85 quaggavt
	useradd -u 92 -g 92 -M -r -s /sbin/nologin
	   -c "Quagga routing suite" -d /var/run/quagga quagga
[On Debian / Ubuntu]
	addgroup --system --gid 92 quagga
	addgroup --system --gid 85 quaggavty
	adduser --system --ingroup quagga --home /var/run/quagga/ \
	   --gecos "Quagga routing suite" --shell /bin/false quagga
	   
groupadd quagga
useradd quagga -g quagga

chown quagga:quagga /var/run/
chmod 777 /var/run/   
chown quagga:quagga /usr/etc/ 编辑配置文件 
chmod 777 /usr/etc/
