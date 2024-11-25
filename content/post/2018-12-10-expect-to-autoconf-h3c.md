---
author: HKL
categories:
- 默认分类
cid: 61
date: "2018-12-10T18:47:00Z"
slug: expect-to-autoconf-h3c
status: publish
tags:
- Coding
- Network
title: 通过expect脚本在H3C设备批量执行命令
updated: 2019/01/29 16:21:11
---


1.本文主要记录了在Linux系统中使用自动化测试工具expect通过ssh登陆H3C设置并批量执行相同命令
	   
2.安装expect

以ubuntu为例

```bash
sudo apt install expect
```

3.编辑expect脚本

<!--more-->
以ssh登陆H3C设置并配置AAA的服务为例

`expect.sh`

```bash
#!/usr/bin/expect
set timeout 5
set f [open ip.txt]   #ip.txt为同目录下配置交换机ip地址的文件
while {1} {
	set ip [gets $f]
	if {[eof $f]} {
		close $f
		break
	}

	spawn ssh -c aes128-cbc -oStrictHostKeyChecking=no YourUsername@$ip
	#expect "Please type 'yes' or 'no'"
	#send "yes\r"
	expect "*password:"
	send "YourPassword\r" #password123为ssh的登陆密码
	expect "*>"
	send "sys\r"
	expect "*]"
	send "radius scheme Schema_aaa\r"
	expect "*a]"
	send "primary authentication YourRadiusPrimaryIP\r"
	expect "*a]"
	send "primary accounting YourRadiusPrimaryIP\r"
	expect "*a]"
	send "secondary authentication YourRadiusSecondaryIP\r"
	expect "*a]"
	send "secondary accounting YourRadiusSecondaryIP\r"
	expect "*a]"
	send "quit\r"
	expect "*]"
	send "quit\r"
	expect "*>"
	send "save\r"
	expect "*Y/N]"
	send "Y\r"
	expect "*):"
	send "\r"
	expect "*Y/N]"
	send "Y\r"
	expect "*>"
	send "quit\r"
}
interact
```

其中`ip.txt`以如下为例

```ini
10.1.1.1
10.1.1.2
10.1.1.3
10.1.1.4
```
