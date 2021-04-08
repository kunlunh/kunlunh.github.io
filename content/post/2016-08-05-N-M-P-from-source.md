---
author: HKL
categories:
- 默认分类
cid: 30
date: "2016-08-05T08:03:00Z"
slug: N-M-P-from-source
status: publish
tags:
- Linux
- Opensource
- Firefox
- Office
title: 从源码编译N(ginx)+M(ySQL)+P(HP)并安装WordPress
updated: 2019/01/29 16:37:06
---


实习项目

过程

**1.下载<a href="http://nginx.org/en/download.html">nginx-1.11.3</a>源码包并且编译。**

因为nginx HTTP rewrite module 需要PCRE包，同时其依赖的zlib本机也没有安装。所以一并下载源码包编译。

其中遇到过一次错误，就是nginx依赖的是PCRE而不是PCRE2，在下载的时候需要区分清楚。

PCRE和zlib编译安装过程比较简单，只需./configure好然后make &amp;&amp; make install即可。

然后可以进入nginx编译过程。

在./configure时需要添加好参数 --with-pcre=PATH/TO/pcre  --with-zlib=PATH/TO/zlib

执行make &amp;&amp; make install 编译安装
<!--more-->
因为没有设定路径，所以安装在/usr/local/nginx目录

修改conf目录下的nginx.conf文件

要对公网开放，所以不能只监听localhost，将listen改成0.0.0.0:80

经过测试，发现也是只有通过本地loop能访问，公网其它主机依然不能访问。初步判断是防火墙设定问题，为iptables添加80端口的ACCEPT规则之后问题解决。

```bash
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```

**2.下载php 7.0.9源码包并进行编译**

因为从php7开始，`./configure`参数中已经删除了"--with-mysql"，换成了"--enable-mysql"，所以`./configure` 添加了`--enable-mysql` 和 `--enable-fpm`

然后编译安装

安装后需要修改`/usr/local/etc`下的`php-fpm.conf`，将`include`参数设定为`php-fpm.d`下的所有.conf文件，并且修改`php-fpm.d`目录下的`www.conf`

再通过修改nginx配置文件`nignx.conf`以支持fastcgi，期间遇到错误就是通过URL访问php文件时没有执行而是直接下载。通过查阅配置文件以及文档，确定是nginx配置文件中fastcgi中的项参数`SCRIPT_FILENAME`配置不正确，将其修改成绝对路径+脚本名字的形式/usr/local/nginx/html$fastcgi_script_name问题解决。

**3.编译安装MySQL5.7.14**

MySQL的编译安装应该是最麻烦的一步，因为本机已经运行了一个mysqld实例，所以需要额外设定一些参数以避免冲突。

首先，编译安装MySQL需要使用cmake，所以先编译安装好cmake，过程比较简单，不过要执行的是`./bootstrap` 而不是`./configure` `make`过程使用`gmake` &amp;&amp; `gmake install`

然后编译安装好依赖包ncurses即可开始配置MySQL

第一次编译安装的时候就是因为参数配置失误，使得安装之后执行mysql遇到了`"segmentation fault (core dumped)"`的问题。第一次是因为只设定了`<span class="pun">-</span><span class="pln">DDEFAULT_CHARSET</span><span class="pun">=</span><span class="pln">utf8mb4`而没有设定`DDEFAULT_COLLATION`参数，在`initialize`过程中遇到`"collation 'latin1_swedish_ci' is not valid for character set 'utf8mb4'"`错误，然后强行修改`my.cnf`文件之后`initialize`通过，能开启mysqld服务，但是客户端每次登录都会报错。</span>

所以第二次重装编译安装，将参数设定好。
```bash
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql57 \
-DMYSQL_UNIX_ADDR=/usr/local/mysql5714/mysql57.sock \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DMYSQL_DATADIR=/var/mysql/data \
-DMYSQL_TCP_PORT=23306 \
-DWITH_BOOST=boost
```
然后mysqld以及mysql均正常。

**4.安装WordPress**

本来导师要求是要安装知识库类的php软件，但是发现似乎这类型的软件还不支持PHP7.0 ，所以只好安装支持PHP7.0的WordPress了。

安装前先在MySQL中新建wordpress数据库，并且授权好

因为MySQL使用的不是默认的3306端口，所以需要修改wp-config.php文件，将

/** MySQL主机 **/一项直接修改成socket口
```php
define('DB_HOST', 'localhost:/usr/local/mysql5714/mysql57.sock');
```
然后访问install.php安装

最终效果：本站

&nbsp;

参考：

<a href="http://java-zone.org/1201.html">1.WordPress安装不使用MySQL数据库默认端口(MySQL默认端口3306)</a>

<a href="https://www.insp.top/article/make-install-mysql-5-7">2.MySQL5.7 的编译安装</a>

<a href="http://wptw.org/create-the-wordpress-database-and-user/">3.建立 MySQL 資料庫</a>

<a href="http://www.tuicool.com/articles/jqIb22">4.lnmp（linux+nginx+mysql+php）源码安装</a>

等等