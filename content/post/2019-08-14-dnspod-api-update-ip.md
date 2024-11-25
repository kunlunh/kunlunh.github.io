---
author: HKL
categories:
- 默认分类
date: "2019-08-14T12:48:00Z"
slug: dnspod-api-update-ip
status: publish
tags:
- Hardware
- Network
title: DNSPOD自动更新公网IP脚本
---


通过DNSPOD提供的API实现自动更新域名公网ip




配置脚本

`cat update_ip.sh`

```bash
#!/bin/sh
ipaddr=`curl -s https://ip.cn | jsonfilter -e "$.ip"`
echo $ipaddr
curl -X POST https://dnsapi.cn/Record.Modify -d "login_token=ID,TOKEN&format=json&domain_id=DOMAIN_ID&record_id=RECORDID&sub_domain=sub&value=$ipaddr&record_type=A&record_line=默认"

```
定时任务

```bash
sudo crontab -l
```


`*/1 * * * * sh /root/update_ip.sh`


获取`domain_id`信息的脚本

```bash
curl -k https://dnsapi.cn/Domain.List -d "login_token=ID,TOKEN"
```

<!--more-->


获取`record_id`信息的脚本

```bash
curl -X POST https://dnsapi.cn/Record.List -d 'login_token=ID,TOKEN&format=json&domain_id=DOMAIN_ID&sub_domain=dormpy&record_type=A&offset=0&length=3'
```
