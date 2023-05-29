---
author: HKL
categories:
- 默认分类
date: "2021-05-30T19:58:00+08:00"
slug: cf-ddns-weixin-push
status: publish
tags:
- Linux
- Operating
- Networking
title: OpenWRT一脚本通过Cloudfalre API更新IP并推送至企业微信
---

本文主要介绍OpenWRT路由器通过Cloudfalre API更新IP并推送至企业微信，单脚本执行。

<!--more-->

脚本代码如下

```bash
#!/bin/sh
wanip_v4=`curl -s -4 ip.sb`    #通过ip.sb提供的服务查询本地出口IPv4地址，有多种实现
record_name='domain.example.com'    #需要在CF更新的域名
zoneid='bfxxxxxxxxxxxxxxxxxxxxx9a2d'    #你的域名zone 在CF的ID
cftoken='BZexxxxxxxxxxxxxxxxxxxxxxxP'    #你申请的域名API TOKEN
zoneinfo=`curl -s -X GET "https://api.cloudflare.com/client/v4/zones/$zoneid/dns_records?name=$record_name&type=A" \
     -H "Authorization: Bearer $cftoken" \
     -H "Content-Type:application/json"`
echo $zoneinfo > /root/hkl/res.json
recordid=`jsonfilter -i /root/hkl/res.json -e '$.result[0].id'`
echo $recordid

result_cf=`curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$zoneid/dns_records/$recordid" \
     -H "Authorization: Bearer $cftoken" \
     -H "Content-Type: application/json" \
     --data "{\"type\":\"A\",\"name\":\"$record_name\",\"content\": \"$wanip_v4\", \"ttl\":1,\"proxied\":false}"`
echo $result_cf  > /root/hkl/cf.json

newipaddr=`jsonfilter -i /root/hkl/cf.json -e '$.result.content'`

wxpushcontent='New IP addr '$newipaddr

corp_id='wwcxxxxxxxxx'    #你的企业微信企业ID
app_secret='xxxxxxxxx'    #你的企业微信应用secret
app_id='100000X'    #你的企业微信应用ID
msg_token=`curl -s -X GET "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=$corp_id&corpsecret=$app_secret"`
echo $msg_token > wx.json

wx_token=`jsonfilter -i /root/hkl/wx.json -e '$.access_token'`
result_wx=`curl -s -X POST "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$wx_token" \
     -H "Content-Type: application/json" \
     --data "{\"touser\":\"@all\",\"agentid\":\"$app_id\",\"text\": {\"content\":\"$wxpushcontent\"}, \"msgtype\":\"text\",\"safe\":0}"`

echo $result_wx

```

然后通过crontab根据需要调用此脚本即可。
