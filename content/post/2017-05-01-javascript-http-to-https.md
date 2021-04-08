---
author: HKL
categories:
- 默认分类
cid: 45
date: "2017-05-01T09:04:00Z"
slug: javascript-http-to-https
status: publish
tags:
- Coding
- Reading
title: 通过JavaScript实现HTTP到HTTPS的强制跳转
updated: 2019/01/29 16:27:56
---


最近通过一些在线文件云空间测试云存储部署静态网站遇到HTTP到HTTPS的强制跳转的问题，平时通过nginx配置是比较简单实现的，
但是例如七牛云虽然可以设置HTTPS访问，但是不支持设置HTTP到HTTPS的强制跳转，
解决方法可以是先通过设置HSTS，通过浏览器级的强制跳转实现，但是本方法只能在webkit内核的浏览器上生效，而且由一定的时间差，
所以最后寻找到通过前端JavaScript脚本实现HTTP到HTTPS的强制跳转，代码如下


```javascript
<script type="text/javascript">
	var targetProtocol = "https:";
	if (window.location.protocol != targetProtocol)
		window.location.href = targetProtocol +
		window.location.href.substring(window.location.protocol.length);
</script>
```

一般将代码防止`<head>` `</head>`之间，这样就能先于body提前加载并作跳转