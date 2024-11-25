---
author: HKL
categories:
- 默认分类
cid: 31
date: "2016-08-09T11:04:00Z"
slug: rsync-custom-ssh-port
status: publish
tags:
- Linux
- Opensource
- Firefox
title: ssh端口更改后rsync的用法
updated: 2019/01/29 17:35:48
---


rsync有两种常用的认证方式，一种为rsync-daemon方式，另外一种则是ssh。
在一些场合，使用rsync-daemon方式会比较缺乏灵活性，ssh方式则成为首选。但是今天实际操作的时候发现当远端服务器的ssh默认端口被修改后，rsync时找不到一个合适的方法来输入对方ssh服务端口号。
在查看官方文档后，找到一种方法，即使用-e参数。
-e参数的作用是可以使用户自由选择欲使用的shell程序来连接远端服务器，当然也可以设置成使用默认的ssh来连接，但是这样我们就可以加入ssh的参数了。
具体语句写法如下：

```bash
rsync -e 'ssh -p 1234' username@hostname:SourceFile DestFile`
```


<!--more-->


其他参数完全按照rsync的规定格式加入即可。
上面语句中比较新鲜的地方就是使用了单引号，目的是为了使引号内的参数为引号内的命令所用。没有引号的话系统就会识别-p是给rsync的一个参数了。我的描述可能比较烂，详情可以参考rsync官方描述：

>Command-line arguments are permitted in COMMAND provided that COMMAND is presented to rsync as a single argument. You must use spaces (not tabs or other whitespace) to separate the command and args from each other, and you can use single- and/or double-quotes to preserve spaces in an argument (but not backslashes). Note that doubling a single-quote inside a single-quoted string gives you a single-quote; likewise for double-quotes (though you need to pay attention to which quotes your shell is parsing and which quotes rsync is parsing).

转自：https://www.centos.bz/2013/09/ssh-port-rsync/
