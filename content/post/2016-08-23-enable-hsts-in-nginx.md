---
author: HKL
categories:
- 默认分类
cid: 37
date: "2016-08-23T09:19:00Z"
slug: enable-hsts-in-nginx
status: publish
tags:
- Operating
- Linux
- Opensource
- Firefox
title: 在nginx中启用HSTS
updated: 2019/01/29 16:33:25
---


从网上查阅资料可知，HTTP 严格传输安全（HSTS）是一种安全功能，web 服务器通过它来告诉浏览器仅用 HTTPS 来与之通讯，而不是使用 HTTP。

而且其实现方法也是比较简单，只需要在http报文的头部增加一项`Strict-Transport-Security`字段就可以了。

相对于原来的将http请求通过301重定向到https相比，

>
>    HSTS 可以用来抵御 SSL 剥离攻击。SSL 剥离攻击是中间人攻击的一种，由 Moxie Marlinspike 于2009年发明。他在当年的黑帽大会上发表的题为 “New Tricks For Defeating SSL In Practice” 的演讲中将这种攻击方式公开。SSL剥离的实施方法是阻止浏览器与服务器创建HTTPS连接。它的前提是用户很少直接在地址栏输入https://，用户总是通过点击链接或3xx重定向，从HTTP页面进入HTTPS页面。所以攻击者可以在用户访问HTTP页面时替换所有https://开头的链接为http://，达到阻止HTTPS的目的。

>    HSTS可以很大程度上解决SSL剥离攻击，因为只要浏览器曾经与服务器创建过一次安全连接，之后浏览器会强制使用HTTPS，即使链接被换成了HTTP。

>   另外，如果中间人使用自己的自签名证书来进行攻击，浏览器会给出警告，但是许多用户会忽略警告。HSTS解决了这一问题，一旦服务器发送了HSTS字段，用户将不再允许忽略警告。


所以在本站的nginx中也添加了这个功能，实现只需要通过在https的`server{ }`段里增加一项`add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";`即可，

<!--more-->

不过需要注意的是，`Strict-Transport-Security`只作用在HTTPS段中，第一次加入通过http请求访问网站，仍旧需要添加一个重定向才可以。

当然，现在谷歌在浏览器安全方面总是走在前面，因此它维护了一个[预载入列表](https://hstspreload.appspot.com/) (请先科学上网)给 Chrome 使用，这个列表会硬编码到 Chrome 浏览器中。后来，Firefox、Safari、 IE 11 和 Edge 也加入了。所以，各大浏览器都支持同一个列表了。这样以来加入到这个列表当中的网站就可以确保在任何情况下都走https了，包括首次。

完整配置如下：

```nginx
server {
	listen 443;
	ssl on;
	ssl_certificate  /path/to/cer;
	ssl_certificate_key  /path/to/key;
	ssl_session_timeout 5m;
	ssl_protocols TLSv1.2 TLSv1.1 TLSv1;
	ssl_ciphers "EECDH+CHACHA20 EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA RC4 !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS !RC4";
	ssl_prefer_server_ciphers on;
        server_name ppuu.org;
	location / {
                root /path/to/index;
                index index.html;
        }
	error_page 404 /404.html;
	location = /404.html {
		root /path/to/404;
		internal;
	}
        add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";
}
server {
	listen 80;
	server_name ppuu.org;
	rewrite ^(.*) https://$server_name$1 permanent;
}
```