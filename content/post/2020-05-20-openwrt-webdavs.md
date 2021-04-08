---
author: HKL
categories:
- 默认分类
date: "2020-05-20T04:12:00Z"
slug: openwrt-webdavs
status: publish
tags:
- Networking
- Operating
title: OpenWRT配置Webdav(s)共享文件
---

本文主要实现在OpenWRT路由器配置Webdav(s)共享文件，主要通过lighttpd

lighttpd版本的webdav可能有些webdav方法不一定支持，需要全功能的webdav可以参考另外一篇用openwrt-apache做的webdav服务器 [/2020/05/openwrt-apache-webdavs/](/2020/05/openwrt-apache-webdavs/)

（1）安装相关软件

```bash
opkg install lighttpd lighttpd-mod-webdav lighttpd-mod-auth lighttpd-mod-authn_file lighttpd-mod-openssl
```


（2）配置lighttpd

以下为模板


<!--more-->

`/etc/lighttpd/lighttpd.conf`
```bash
server.document-root        = "/mnt/sda1/files"
server.upload-dirs          = ( "/tmp" )
server.errorlog             = "/var/log/lighttpd/error.log"
server.pid-file             = "/var/run/lighttpd.pid"
server.username             = "http"
server.groupname            = "www-data"

index-file.names            = ( "index.php", "index.html",
                                "index.htm", "default.htm",
                              )


$SERVER["socket"] == "0.0.0.0:443" {
	ssl.engine = "enable"
	ssl.pemfile = "/mnt/sda1/etc/ca/fullchain.cer"
		ssl.privkey= "/mnt/sda1/etc/ca/domain.key"
}

static-file.exclude-extensions = ( ".php", ".pl", ".fcgi" )

### Options that are useful but not always necessary:
#server.chroot               = "/"
server.port                 = 81
#server.bind                 = "localhost"
#server.tag                  = "lighttpd"
server.errorlog-use-syslog  = "enable"
#server.network-backend      = "writev"

### Use IPv6 if available
#include_shell "/usr/share/lighttpd/use-ipv6.pl"

dir-listing.encoding        = "utf-8"
server.dir-listing          = "enable"

include "/etc/lighttpd/mime.conf"
include "/etc/lighttpd/conf.d/*.conf"
```

`/etc/lighttpd/conf.d/20-auth.conf`
```bash
#######################################################################
##
##  Authentication Module
## -----------------------
##
## See https://redmine.lighttpd.net/projects/lighttpd/wiki/docs_modauth
## for more info.
##
server.modules += ( "mod_auth" )

auth.backend                 = "plain"
auth.backend.plain.userfile  = "/mnt/sda1/etc/webdav/lighttpd.user"
#auth.backend.plain.groupfile = "/etc/lighttpd/lighttpd.group"

#auth.backend.ldap.hostname = "localhost"
#auth.backend.ldap.base-dn  = "dc=my-domain,dc=com"
#auth.backend.ldap.filter   = "(uid=$)"

auth.require               = ( "/" =>
                               (
                                 "method"  => "basic",
                                 "realm"   => "Webdav Server",
                                 "require" => "valid-user"
                               ),
                             )

##
#######################################################################
```
lighttpd.user为明文用户名密码形式：

```
admin1:password1
admin2:password2
```

`/etc/lighttpd/conf.d/20-auth.conf`
```bash
#######################################################################
##
##  WebDAV Module
## ---------------
##
## See https://redmine.lighttpd.net/projects/lighttpd/wiki/Docs_ModWebDAV
##
server.modules += ( "mod_webdav" )

#$HTTP["url"] =~ "^/dav($|/)" {
  ##
  ## enable webdav for this location
  ##
  webdav.activate = "enable"

  ##
  ## By default the webdav url is writable.
  ## Uncomment the following line if you want to make it readonly.
  ##
  #webdav.is-readonly = "enable"

  ##
  ## Log the XML Request bodies for debugging
  ##
  #webdav.log-xml = "disable"

  ##
  ##
  ##
  webdav.sqlite-db-name = "/mnt/sda1/etc/webdav/webdav.db"
#}
##
#######################################################################
```

启用全局Webdav，因此注释了原始文件的站点配置

（3）启用

```bash
/etc/init.d/lighttpd enable
/etc/init.d/lighttpd start
```
