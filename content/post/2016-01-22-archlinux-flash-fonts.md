---
author: HKL
categories:
- 默认分类
cid: 18
date: "2016-01-22T05:20:00Z"
slug: archlinux-flash-fonts
status: publish
tags:
- Linux
- Opensource
- Office
title: 解决Archlinux系统Flash乱码
updated: 2019/01/29 17:37:31
---


1.安装文泉驿字体
```
    # pacman -S wqy-zhenhei
```

2.用你的编辑器，打开并修改 /etc/fonts/conf.d/49-sansserif.conf ，修改内容为以下：


<!--more-->


```xml
<?xml version="1.0"?>  
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">  
<fontconfig>  
<!--  
  If the font still has no generic name, add sans-serif  
-->  
    <match target="pattern">  
        <test qual="all" name="family" compare="not_eq">  
            <string>sans-serif</string>  
        </test>  
        <test qual="all" name="family" compare="not_eq">  
            <string>serif</string>  
        </test>  
        <test qual="all" name="family" compare="not_eq">  
            <string>monospace</string>  
        </test>  
        <edit name="family" mode="append_last">  
            <string>WenQuanYi Zen Hei</string>  
        </edit>  
    </match>  
</fontconfig>  
```

上面的文件主要修改了倒数第四行，将原来的
```xml
<string>sans-serif</string>
```

改成了 

```xml
<string>WenQuanYi Zen Hei</string>
```
3.保存修改过的 49-sansserif.conf ，重启一下阅览器，应该就可以正常使用 flash 了。


本文转自 http://huangkunlun.cn/2013/08/23/解决archlinux系统flash乱码/
