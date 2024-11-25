---
author: HKL
categories:
- 默认分类
cid: 38
date: "2016-08-30T18:19:00Z"
slug: iis6-mysql-php
status: publish
tags:
- Operating
- Windows
title: 配置IIS6+PHP5.6+MySQL5
updated: 2019/01/29 16:32:54
---


受学院老师所托，从学校学院那边接手了一台托管在网络中心的windows server 2008服务器，并且要部署好php和mysql数据库的环境，虽然平时也是比较喜欢服务器运维的，不过是Linux方向，没怎么接触过windows server。
本来想直接重装个CentOS上去，不过好像以后可能还会有.net或者asp的东西会要用到，申请机房还比较麻烦，所以还是决定使用win server，感觉道理是差不多的，应该没有问题。

之前管理的同学告知我wordpress连不上MySQL数据库，觉得是安装出了问题。然后登上服务器一看，发现原来的服务器充斥着phpstudy、xampp等一键部署的东西，文件乱放，没有统一固定的安置方式，网站源码也是D:\有一个，E:\有一个，
对于平时在linux上都是最小化安装的来说有种不能忍的感觉，而不能连MySQL也就是一个简单的端口占用问题，关启服务就已经正常了，不过觉得这种一键的方式还是很不好，
服务器的可扩展性和可维护性都太差了，所以觉得至少也需要每个服务独立部署，然后再集成。


<!--more-->


Windows Server 这种有图形界面的服务器虽然不同命令行的部署，不过也不是特别难，IIS也觉得还是挺方便的。

首先，还是需要将一键部署的程序先卸载掉了。然后再从新部署各种服务，由于服务器是学校教育网内部的地址，和电信、联通这些ISP都不互通，所以每次安装包都需要从本机下载好再上传过去，所以就选择了最基础的服务进行安装，

MySQL官网上还提供下载的纯Server版本就只有5.5了，而5.5也基本能满足老师交代的东西的，安装过程还是比较简单，安装后通过Instance就可以配置好root帐号和密码，并且启动好服务。（因为之前管理服务器的同学可能也安装过MySQL而且又使用一键部署包，造成一开始卡在Instance配置的`Start Service`里，网上查阅是需要删除注册表的几项数据，删除后重装就正常了）
[mysql-5.5.51-winx64.msi](http://dev.mysql.com/get/Downloads/MySQL-5.5/mysql-5.5.51-winx64.msi)

接着就是部署PHP环境，首先还是需要在官网下载程序，由于7版本很多程序还没有适配过来，所以决定先用5.6的版本，因为准备用IIS作web服务器，需要使用 Non Thread Safe的版本，
[php-5.6.25-nts-Win32-VC11-x64.zip](http://windows.php.net/downloads/releases/php-5.6.25-nts-Win32-VC11-x64.zip)

下载之后解压到C:\PHP\，然后需要修改php.ini以启动php服务和支持MySQL，之后在IIS里面添加PHP的ISAPI路径和配置好FastCGI，通过phpinfo()然后在MySQL里新建好表和用户就可以安装Wordpress了，

这个服务器基本配置就告一段落，以后再按需要搭建一些Java环境给其他项目用了。


参考：

1.[Win2012 R2 IIS8.5+PHP(FastCGI)+MySQL运行环境搭建教程](http://www.jb51.net/article/59280.htm)

2.[iis7.5安装配置php环境详细清晰教程](http://www.webkaka.com/blog/archives/how-to-config-php-environment-in-iis7.5.html)