---
author: HKL
categories:
- 默认分类
cid: 36
date: "2016-08-22T12:19:00Z"
slug: optimize-nginx-jemalloc
status: publish
tags:
- Operating
- Linux
- Opensource
- Firefox
title: 使用jemalloc对nginx进行优化
updated: 2019/01/29 17:34:44
---


jemalloc是比glibc中的malloc高效很多的内存管理方案。

在nginx中也支持使用jemalloc进行内存管理，那更应该一试了。

一、安装jemalloc

```bash
cd /opt/soft
wget https://github.com/jemalloc/jemalloc/releases/download/4.2.1/jemalloc-4.2.1.tar.bz2 -O jemalloc-4.2.1.tar.bz2
tar -xvf jemalloc-4.2.1.tar.bz2
cd jemalloc-4.2.1
./configure
make && make install_bin install_include install_lib
echo '/usr/local/lib' > /etc/ld.so.conf.d/local.conf
ldconfig
```

这样就应该安装完毕了。

二、添加jemalloc tag重新编译nginx


<!--more-->


(主要就是在configure中添加一项`--with-ld-opt=-ljemalloc`)
```bash
cd PATH/TO/NGINX
./configure --prefix=/opt/nginx-1.11.3 --with-http_ssl_module --with-pcre=../pcre-8.39 --with-zlib=../zlib-1.2.8 --with-openssl=../openssl-1.0.2h --with-ipv6 --with-ld-opt=-ljemalloc
make && make install
```

三、验证jemalloc是否正常运行

执行
```bash
lsof -n | grep jemalloc
```

应该会有如下的输出
```bash
nginx     31573   root  mem       REG                8,4            54411954 /usr/local/lib/libjemalloc.so.2 (path dev=244,196)
nginx     31606 nobody  mem       REG                8,4            54411954 /usr/local/lib/libjemalloc.so.2 (path dev=244,196)
```