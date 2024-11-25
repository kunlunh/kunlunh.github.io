---
author: HKL
categories:
- 默认分类
cid: 54
date: "2018-08-02T20:43:00Z"
slug: Unix-monitor-process
status: publish
tags:
- Operating
- FreeBSD
- Linux
title: Unix每分钟监控进程的状态
updated: 2019/01/29 17:32:36
---


（以FreeBSD为服务器监控frps进程为例）

1、创建监控Shell脚本`monfrp.sh`

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
   	/root/frp/frps -c /root/frp/frps.ini && echo "frpc start"
else
echo "the process is running"
fi
```


<!--more-->


2、为monfrp.sh添加执行权限
```bash
chmod +x monfrp.sh
```

3、为monfrp.sh添加定时执行
```bash
crontab -e
```

添加一行

```bash
*/1 * * * * sh /PATH/TO/monfrp.sh
```