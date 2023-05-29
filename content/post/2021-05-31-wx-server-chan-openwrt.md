---
author: HKL
categories:
- 默认分类
date: "2021-05-31T15:20:00+08:00"
slug: wx-serverchan-openwrt
status: publish
tags:
- Linux
- Operating
- Networking
title: OpenWRT简易版serverchan脚本推送至企业微信
---

本文主要介绍简易版serverchan脚本推送至企业微信。

<!--more-->

脚本代码如下

```bash
#!/bin/sh
export PATH='/usr/sbin:/usr/bin:/sbin:/bin'

# Paraments
resub=1
push1="1"				# Push online devices
push_ddns="1"			# Push ddns message
qywxpusher_enable="1"	# Enable ServerChan
#corp_id='wwcxxxxxxxxx'    #你的企业微信企业ID
#app_secret='xxxxxxxxx'    #你的企业微信应用secret
#app_id='100000X'    #你的企业微信应用ID



touch /tmp/tmp/lastIPAddress
[ ! -s /tmp/tmp/lastIPAddress ] && echo "Initialing！" > /tmp/tmp/lastIPAddress

touch /tmp/tmp/pushDevice
[ ! -s /tmp/tmp/pushDevice ] && echo "$push1" > /tmp/tmp/pushDevice

# Get wan IP
# Check curl exist, if not, use wget
getIpAddress() {
    curltest=`which curl`
    if [ -z "$curltest" ] || [ ! -s "`which curl`" ] ; then
        wget --no-check-certificate --quiet --output-document=- "http://members.3322.org/dyndns/getip"
    else
        curl -k -s "http://members.3322.org/dyndns/getip"
    fi
}
# load last IP
lastIPAddress() {
        inter="/tmp/tmp/lastIPAddress"
        cat $inter
}



# Get online devices
test(){
	alias=`cat /root/devicelist`
        cat /proc/net/arp | grep -v "Mask" | sed 's/(//;s/)//' | while read -r IP HW FLAGS MAC MASK DEVICE
        do
        if [ $DEVICE == "br-lan" ]; then
            #echo $DEVICE
                NAME=`echo "$alias" | awk '/'$MAC'\ '$IP'/{print $4}'`
            if [ ! -n "$NAME" ]; then
                    NAME="Unknown"
            else
                    NAME=${NAME}
            fi
            echo $NAME $IP >> /tmp/tmp/newhostname.txt
            #echo $NAME $IP
        fi
        done
~

}


while [ "$qywxpusher_enable" == "1" ];
do
curltest=`which curl`
if [ -z "$curltest" ] ; then
    wget --continue --no-check-certificate  -q -T 10 http://www.baidu.com
	[ "$?" == "0" ] && check=200 || check=404
else
    check=`curl -k -s -w "%{http_code}" "http://www.baidu.com" -o /dev/null`
fi

if [ "$check" == "200" ] ; then
hostIP=$(getIpAddress)
lastIP=$(lastIPAddress)

if [ "$lastIP" != "$hostIP" ] && [ ! -z "$hostIP" ] ; then
	sleep 60
    # Check again
	hostIP=$(getIpAddress)
    lastIP=$(lastIPAddress)
 fi

if [ "$lastIP" != "$hostIP" ] && [ ! -z "$hostIP" ] ; then
    logger -t "公网IP变动" "目前 IP: ${hostIP}"
    logger -t "公网IP变动" "上次 IP: ${lastIP}"
	if [ "$?" == "0" ] ; then
		if [ "$push_ddns" = "1" ] ; then
            wxpushcontent='New WAN IP address: '${hostIP}
            msg_token=`curl -s -X GET "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=$corp_id&corpsecret=$app_secret"`
            echo $msg_token > /tmp/tmp/wx.json
            wx_token=`jsonfilter -i /tmp/tmp/wx.json -e '$.access_token'`
            curl -s -X POST "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$wx_token" \
               -H "Content-Type: application/json" \
              --data "{\"touser\":\"@all\",\"agentid\":\"$app_id\",\"text\": {\"content\":\"$wxpushcontent\"}, \"msgtype\":\"text\",\"safe\":0}" &
			logger -t "wechat push" "IP: ${hostIP} pushed"
		fi
		echo -n $hostIP > /tmp/tmp/lastIPAddress
	fi
fi
if [ `cat /tmp/tmp/pushDevice` = "1" ] ; then
    # 设备上、下线提醒
    # 获取接入设备名称
    touch /tmp/tmp/newhostname.txt
    echo "接入设备名称" > /tmp/tmp/newhostname.txt	
    # 当前所有接入设备
	test
	# 读取已在线设备名称
    touch /tmp/tmp/hostname_online.txt
    [ ! -s /tmp/tmp/hostname_online.txt ] && echo "接入设备名称" > /tmp/tmp/hostname_online.txt
    # 上线
    awk 'NR==FNR{a[$0]++} NR>FNR&&a[$0]' /tmp/tmp/hostname_online.txt /tmp/tmp/newhostname.txt > /tmp/tmp/newhostname_same_online.txt
    awk 'NR==FNR{a[$0]++} NR>FNR&&!a[$0]' /tmp/tmp/newhostname_same_online.txt /tmp/tmp/newhostname.txt > /tmp/tmp/newhostname_uniqe_online.txt
    if [ -s "/tmp/tmp/newhostname_uniqe_online.txt" ] ; then
		content=`cat /tmp/tmp/newhostname_uniqe_online.txt | grep -v "^$"`
        onlineDev=`cat /tmp/tmp/hostname_online.txt | grep -v "^$"`
        wxpushcontent='设备上线通知\n------------------\n'${content}'\n\n------------------\n'${onlineDev}
        msg_token=`curl -s -X GET "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=$corp_id&corpsecret=$app_secret"`
            echo $msg_token > /tmp/tmp/wx.json
            wx_token=`jsonfilter -i /tmp/tmp/wx.json -e '$.access_token'`
            curl -s -X POST "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$wx_token" \
               -H "Content-Type: application/json" \
              --data "{\"touser\":\"@all\",\"agentid\":\"$app_id\",\"text\": {\"content\":\"$wxpushcontent\"}, \"msgtype\":\"text\",\"safe\":0}" &

		logger -t "wechat push" "设备上线: ${content} pushed"
		cat /tmp/tmp/newhostname_uniqe_online.txt | grep -v "^$" >> /tmp/tmp/hostname_online.txt
    fi
    # 下线
    awk 'NR==FNR{a[$0]++} NR>FNR&&!a[$0]' /tmp/tmp/newhostname.txt /tmp/tmp/hostname_online.txt > /tmp/tmp/newhostname_uniqe_offline.txt
    if [ -s "/tmp/tmp/newhostname_uniqe_offline.txt" ] ; then
       content=`cat /tmp/tmp/newhostname_uniqe_offline.txt | grep -v "^$"`
       onlineDev=`cat /tmp/tmp/hostname_online.txt | grep -v "^$"`
       wxpushcontent='设备下线通知\n------------------\n'${content}'\n\n------------------\n'${onlineDev}
        msg_token=`curl -s -X GET "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=$corp_id&corpsecret=$app_secret"`
            echo $msg_token > /tmp/tmp/wx.json
            wx_token=`jsonfilter -i /tmp/tmp/wx.json -e '$.access_token'`
            curl -s -X POST "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$wx_token" \
               -H "Content-Type: application/json" \
              --data "{\"touser\":\"@all\",\"agentid\":\"$app_id\",\"text\": {\"content\":\"$wxpushcontent\"}, \"msgtype\":\"text\",\"safe\":0}" &

       logger -t "wechat push" "设备下线: ${content} pushed"
       cat /tmp/tmp/newhostname.txt | grep -v "^$" > /tmp/tmp/hostname_online.txt
    fi
fi
resub=`expr $resub + 1`
[ "$resub" -gt 360 ] && resub=1
else
logger -t "wxpusher service" "Check network failed."
resub=1
fi
sleep 30
continue
done

```

把上面的代码保存在/root/serverchan.sh

再在管理界面-系统-startup最下面位置的脚本块的结尾加入一行

(/root/serverchan.sh) &

或者其他方式随系统启动脚本就成功了。