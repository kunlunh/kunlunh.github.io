---
author: HKL
categories:
- 默认分类
cid: 34
date: "2016-08-17T17:34:00Z"
slug: haproxy-with-nginx
status: publish
tags:
- Operating
- Linux
- Firefox
title: haproxy与nginx集成实例
updated: 2019/01/29 17:35:03
---


一、nginx的配置文件(主要部分)如下：

`server1.conf`

```nginx
server {
    listen       8189;
    server_name  127.0.0.1;
    root   /opt/nginx/html/php;
    index  index.html index.htm index.php;

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
       
}
```

`server2.conf`


<!--more-->


```nginx
server {
    listen       8189;
    server_name  127.0.0.1;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    root   /opt/nginx/html/php;
    index  index.html index.htm index.php;
     
    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
       
}
```

```bash
[root@test conf.d]# cat wp.conf
```

```nginx
server {
    listen       8188;
    server_name 127.0.0.1;
    root /opt/nginx/html;
    index  index.html index.htm index.php;

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
    #include global/restrictions.conf;
    #include global/wordpress.conf;
}
```

主要是将原来bind的server_name 0.0.0.0、端口都换成本地地址和端口，以腾出给haproxy使用。

二、haproxy配置文件(主要部分)如下：
只需要根据客户端请求的url中匹配目录的名称即可。具体配置文件如下：
```apacheconf
global
	log 127.0.0.1 local0
	log 127.0.0.1 local1 notice
	maxconn 4096
	uid 188
	gid 188
	daemon
defaults
	log global
	mode http
	option httplog
	option dontlognull
	option http-server-close
	option forwardfor except 127.0.0.1
	option redispatch
	retries 3
	option redispatch
	maxconn 2000
	timeout http-request 10s
	timeout queue 1m
	timeout connect 10s
	timeout client 1m
	timeout server 1m
	timeout http-keep-alive 10s
	timeout check 10s
	maxconn 3000
listen admin_stats
	bind 0.0.0.0:10801
	mode http
	option httplog
	maxconn 10
	stats refresh 30s
	stats uri /stats
	stats auth admin:admin
	stats hide-version
frontend weblb
	bind *:80
	acl is_nginx hdr_beg(host) test.ppuu.org
	acl is_http hdr_beg(host) 42.123.77.14
	use_backend nginxserver if is_nginx 
	use_backend httpserver if is_http
backend httpserver
	balance source
	server web1 127.0.0.1:8188 maxconn 1024 weight 3 check inter 2000 rise 2 fall 3
backend nginxserver
	balance source
	server web1 127.0.0.1:8189 maxconn 1024 weight 3 check inter 2000 rise 2 fall 3

```

三、效果
通过访问[http://42.123.77.14](http://42.123.77.14)
以及[http://test.ppuu.org](http://test.ppuu.org)
可得出相应的效果。