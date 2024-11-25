---
author: HKL
categories:
- 默认分类
date: "2019-08-16T06:11:00Z"
slug: ngrokc-prebuild-for-ramips
status: publish
tags:
- Hardware
- Network
title: ngrokc rampis预编译版本
---

[ngrokc](https://github.com/dosgo/ngrok-c) 是用c语言实现的ngrok1的客户端，非常适合在嵌入式设备中使用，

不过官方github上面的release已经有点旧了，在18.06.4上已经无法正常运行，所以重新用最新的18.06.4 ramips SDK编译，测试过mt7620和mt7621的机器都能可以正常运行。

tested on mt7620 and mt7621

下载链接：

[https://stu2013jnueducn-my.sharepoint.com/:u:/g/personal/hkl_stu2013_jnu_edu_cn/EbcXrXWPY4pJpXcLWh_XoKEBkY2D3anyCiOLo9qAax6PeQ?e=OXNypn](https://stu2013jnueducn-my.sharepoint.com/:u:/g/personal/hkl_stu2013_jnu_edu_cn/EbcXrXWPY4pJpXcLWh_XoKEBkY2D3anyCiOLo9qAax6PeQ?e=OXNypn)



使用说明：

<!--more-->

先安装依赖

`opkg install libopenssl libstdcpp`


运到方式

```bash
./ngrokc
use ./ngrokc -SER[Shost:ngrokd.ngrok.com,Sport:443,Atoken:xxxxxxx,Password:xxx] -AddTun[Type:tcp,Lhost:127.0.0.1,Lport:80,Rport:50199,Hostheader:localhost]
```
