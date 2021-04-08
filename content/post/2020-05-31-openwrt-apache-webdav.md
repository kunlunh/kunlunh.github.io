---
author: HKL
categories:
- 默认分类
date: "2020-05-31T13:12:00Z"
slug: openwrt-apache-webdavs
status: publish
tags:
- Networking
- Operating
title: OpenWRT配置Apache Webdav
---

本文主要实现在OpenWRT路由器配置Apache2 based 的 Webdav(s)共享文件，之前曾经试过通过 [lighttpd部署Webdav](/2020/05/openwrt-webdavs/) 不过由于在尝试通过lighttpd部署的Webdav作为[Joplin](https://joplinapp.org/)的后端Webdav存储时，会出现4XX的故障码，经查询，应该是lighttpd的Webdav默认不是全部的Webdav Method都支持，所以这次改用OpenWRT Apache2 Webdav


（1）安装相关软件

```bash
opkg install apache2 apache-mod-webdav apache-mod-ssl
```


（2）配置apache2

以下为模板
由于这次部署基本打算也是全站开启webdav,所以以vhost模式走webdav

<!--more-->

主配置文件，基本保持默认，添加监听端口以及认证的Module，去掉注释即可，都是OpenWRT安装好Apache默认配置文件会有的Module

`/etc/apache2/apache2.conf`
```bash
ServerRoot "/usr"

Listen 81 #4443为Webdav端口
Listen 4443 #4443为Webdav-ssl端口

LoadModule mpm_prefork_module lib/apache2/mod_mpm_prefork.so
LoadModule authn_file_module lib/apache2/mod_authn_file.so
LoadModule authn_core_module lib/apache2/mod_authn_core.so
LoadModule authz_host_module lib/apache2/mod_authz_host.so
LoadModule authz_groupfile_module lib/apache2/mod_authz_groupfile.so
LoadModule authz_user_module lib/apache2/mod_authz_user.so
LoadModule authz_core_module lib/apache2/mod_authz_core.so
LoadModule access_compat_module lib/apache2/mod_access_compat.so
LoadModule auth_basic_module lib/apache2/mod_auth_basic.so
LoadModule auth_digest_module lib/apache2/mod_auth_digest.so
LoadModule socache_shmcb_module lib/apache2/mod_socache_shmcb.so
LoadModule reqtimeout_module lib/apache2/mod_reqtimeout.so
LoadModule filter_module lib/apache2/mod_filter.so
LoadModule mime_module lib/apache2/mod_mime.so
LoadModule log_config_module lib/apache2/mod_log_config.so
LoadModule env_module lib/apache2/mod_env.so
LoadModule headers_module lib/apache2/mod_headers.so
LoadModule setenvif_module lib/apache2/mod_setenvif.so
LoadModule version_module lib/apache2/mod_version.so
LoadModule ssl_module lib/apache2/mod_ssl.so
LoadModule unixd_module lib/apache2/mod_unixd.so
LoadModule dav_module lib/apache2/mod_dav.so
LoadModule status_module lib/apache2/mod_status.so
LoadModule autoindex_module lib/apache2/mod_autoindex.so
<IfModule !mpm_prefork_module>
        #LoadModule cgid_module lib/apache2/mod_cgid.so
</IfModule>
<IfModule mpm_prefork_module>
        #LoadModule cgi_module lib/apache2/mod_cgi.so
</IfModule>
LoadModule dav_fs_module lib/apache2/mod_dav_fs.so
LoadModule dav_lock_module lib/apache2/mod_dav_lock.so
LoadModule dir_module lib/apache2/mod_dir.so
LoadModule alias_module lib/apache2/mod_alias.so

<IfModule unixd_module>
User apache
Group apache

</IfModule>

ServerName 127.0.0.1:81

<Directory />
    AllowOverride none
    Require all denied
</Directory>

DocumentRoot "/usr/share/apache2/htdocs"
<Directory "/usr/share/apache2/htdocs">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>

<Files ".ht*">
    Require all denied
</Files>

ErrorLog "/var/log/apache2/error_log"

LogLevel warn

<IfModule log_config_module>

    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common

    <IfModule logio_module>
      # You need to enable mod_logio.c to use %I and %O
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>

    CustomLog "/var/log/apache2/access_log" common

</IfModule>

<IfModule alias_module>

    ScriptAlias /cgi-bin/ "/usr/share/apache2/cgi-bin/"

</IfModule>

<Directory "/usr/share/apache2/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>

<IfModule headers_module>

    RequestHeader unset Proxy early
</IfModule>

<IfModule mime_module>

    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
</IfModule>

Include /etc/apache2/extra/httpd-vhosts.conf
Include /etc/apache2/extra/httpd-dav.conf

<IfModule proxy_html_module>
Include /etc/apache2/extra/proxy-html.conf
</IfModule>

Include /etc/apache2/extra/httpd-ssl.conf
<IfModule ssl_module>
SSLRandomSeed startup builtin
SSLRandomSeed connect builtin
</IfModule>
```


站点配置文件：

`/etc/apache2/extra/httpd-vhosts.conf`
```bash
<VirtualHost *:81>
    ServerAdmin webmaster@dummy-host.example.com
    DocumentRoot "/mnt/sda1/files"
    DirectoryIndex disabled #Golbal Webdav,如果没有改配置Webdav不能读取Webroot
    <Directory />
    #Options -Indexes +FollowSymlinks
    Options +Indexes
    #AllowOverride All
    #Require all granted
    Dav On

    AuthType Basic  #Joplin的Webdav同步模式只支持Basic
    AuthName "Auth Required"
    #   htdigest -c "/usr/user.passwd" DAV-upload admin
    AuthUserFile "/etc/apache2/user.basic"
    #AuthDigestProvider file
    Require valid-user
    </Directory>
    ServerName dummy-host.example.com
    ServerAlias www.dummy-host.example.com
    ErrorLog "/var/log/apache2/dummy-host.example.com-error_log"
    #CustomLog "/var/log/apache2/dummy-host.example.com-access_log" common
</VirtualHost>
```
`/etc/apache2/user.basic`可以通过`htpasswd -c /etc/apache2/user.basic <USERNAME>`生成：

```
admin1:password1hash
admin2:password1hash
```
```bash
chown -R apache:apache /etc/apache2/user.basic
```

HTTPS站点配置

`/etc/apache2/extra/httpd-ssl.conf`
```bash

SSLCipherSuite HIGH:MEDIUM:!MD5:!RC4:!3DES
SSLProxyCipherSuite HIGH:MEDIUM:!MD5:!RC4:!3DES

SSLHonorCipherOrder on
SSLProtocol all -SSLv3
SSLProxyProtocol all -SSLv3

SSLPassPhraseDialog  builtin

#SSLSessionCache         "dbm:/var/run/apache2/ssl_scache"
SSLSessionCache        "shmcb:/var/run/apache2/ssl_scache(512000)"
SSLSessionCacheTimeout  300

<VirtualHost *:4443>

#   General setup for the virtual host
DocumentRoot "/mnt/sda1/files"
ServerName nsb.fac.pub:4443
ServerAdmin you@example.com
#ErrorLog "/var/log/apache2/error_log"
#TransferLog "/var/log/apache2/access_log"
DirectoryIndex disabled
    <Directory />
    #Options -Indexes +FollowSymlinks
    Options +Indexes
    Dav On
    #Require valid-user
    AuthType Basic
    AuthName "Auth Required"
    #   htdigest -c "/usr/user.passwd" DAV-upload admin
    AuthUserFile "/etc/apache2/user.basic"
    #AuthDigestProvider file
    Require valid-user
    </Directory>
SSLEngine on
SSLCertificateFile "/mnt/sda1/etc/ca/fullchain.cer"
SSLCertificateKeyFile "/mnt/sda1/etc/ca/fac.pub.key"

<FilesMatch "\.(cgi|shtml|phtml|php)$">
    SSLOptions +StdEnvVars
</FilesMatch>
<Directory "/usr/share/apache2/cgi-bin">
    SSLOptions +StdEnvVars
</Directory>

BrowserMatch "MSIE [2-5]" \
         nokeepalive ssl-unclean-shutdown \
         downgrade-1.0 force-response-1.0

</VirtualHost>
```


Webdav相关配置

`/etc/apache2/extra/httpd-dav.conf`
```bash
DavLockDB "/mnt/sda1/etc/webdav/DavLock"

Alias /uploads "/usr/uploads"

<Directory "/usr/uploads">
    Dav On

    AuthType Digest
    AuthName DAV-upload
    # You can use the htdigest program to create the password database:
    #   htdigest -c "/usr/user.passwd" DAV-upload admin
    AuthUserFile "/usr/user.passwd"
    AuthDigestProvider file

    # Allow universal read-access, but writes are restricted
    # to the admin user.
    <RequireAny>
        Require method GET POST OPTIONS
        Require user admin
    </RequireAny>
</Directory>

#
# The following directives disable redirects on non-GET requests for
# a directory that does not include the trailing slash.  This fixes a
# problem with several clients that do not appropriately handle
# redirects for folders with DAV methods.
#
BrowserMatch "Microsoft Data Access Internet Publishing Provider" redirect-carefully
BrowserMatch "MS FrontPage" redirect-carefully
BrowserMatch "^WebDrive" redirect-carefully
BrowserMatch "^WebDAVFS/1.[01234]" redirect-carefully
BrowserMatch "^gnome-vfs/1.0" redirect-carefully
BrowserMatch "^XML Spy" redirect-carefully
BrowserMatch "^Dreamweaver-WebDAV-SCM1" redirect-carefully
BrowserMatch " Konqueror/4" redirect-carefully
```


（3）启用

```bash
/etc/init.d/apache2 enable
/etc/init.d/apache2 start
```


这样的话经测试时可以兼容[Joplin](https://joplinapp.org/)的Webdav存储后端的。

Enjoy!