---
author: HKL
categories:
- 默认分类
date: "2019-08-13T23:35:00Z"
slug: frp-monitor-script
status: publish
tags:
- Hardware
- Network
title: frp定时监控脚本
---

frp是用和比较多的反代了，这种工具目的就是24online的，所以写一个监控脚本让其不断线，或者程序出问题终止后自动运行是很有必要的，特别是在内网IP的主机了，
frp一出问题就再连不上，所以让其本地恢复是相当重要。

脚本用于frp服务器和客户端

<!--more-->

服务器侧

`cat mon_frps.sh`

```bash
#! /bin/sh      
proc_name="frps"        #进程名
       
proc_num()                      #查询进程数量
{
 	num=`ps -ef | grep $proc_name | grep -v grep | wc -l`  #视乎情况"ps -ef"需要更改为"ps -aux"
   	return $num
}
            
proc_num
number=$?                       #获取进程数量
if [ $number -eq 0 ]            #如果进程数量为0
then                            #重新启动服务器，或者扩展其它内容。
   	/home/ubuntu/frp/frps -c /home/ubuntu/frp/frps.ini && echo "frps start"
else
echo "the process is running"
fi
```

```bash
sudo crontab -l
```

`*/1 * * * * sh /home/ubuntu/mon_frps.sh`


客户端

`cat mon_frpc.sh`

```bash
#! /bin/sh      
proc_name="frpc"        #进程名
       
proc_num()                      #查询进程数量
{
 	num=`ps | grep $proc_name | grep -v grep | wc -l`
   	return $num
}
            
proc_num
number=$?                       #获取进程数量
if [ $number -eq 0 ]            #如果进程数量为0
then                            #重新启动服务器，或者扩展其它内容。
   	/usr/bin/frpc -c /root/frpc.ini && echo "frpc start"
else
echo "the process is running"
fi
```

```bash
sudo crontab -l
```


`*/1 * * * * sh /home/ubuntu/mon_frpc.sh`
