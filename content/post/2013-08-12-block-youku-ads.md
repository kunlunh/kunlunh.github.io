---
author: HKL
categories:
- 默认分类
cid: 15
date: "2013-08-12T12:52:00Z"
slug: block-youku-ads
status: publish
tags:
- Blog
- Firefox
title: 彻底屏蔽优酷广告
updated: 2019/01/29 16:45:23
---


不久之前网络上有一个通过修改Hosts来屏蔽各大视频网站广告的方法，谁想到优酷嗅觉灵敏，很快推出了反屏蔽的策略——即便不看广告也会有30秒的黑屏等待。正所谓“道高一尺，魔高一丈”，广大电工又研究出了反“反屏蔽”的方法……


<!--more-->


### 第一步：屏蔽Hosts

在操作系统的Hosts文件（`C:\Windows\System32\drivers\etc\hosts`）中加入如下内容：

```xml
#Youku

127.0.0.1 atm.youku.com

127.0.0.1 fvid.atm.youku.com

127.0.0.1 html.atm.youku.com

127.0.0.1 valb.atm.youku.com

127.0.0.1 valc.atm.youku.com

127.0.0.1 valf.atm.youku.com

127.0.0.1 valo.atm.youku.com

127.0.0.1 valp.atm.youku.com

127.0.0.1 vid.atm.youku.com

127.0.0.1 walp.atm.youku.com

127.0.0.1 lstat.youku.com

127.0.0.1 speed.lstat.youku.com

127.0.0.1 static.lstat.youku.com

127.0.0.1 urchin.lstat.youku.com

127.0.0.1 stat.youku.com
```


### 第二步：屏蔽Flash SharedObjects

首先找到Flash的SharedObjects存储路径，删除“static.youku.com”这个目录：


Windows XP：

`C:\Documents and Settings\Administrator\Application Data\Macromedia\Flash Player\#SharedObjects\[随机数]\static.youku.com`

Windows 7：

`C:\Users\Administrator\AppData\Roaming\Macromedia\Flash Player\#SharedObjects\[随机数]\static.youku.com`

然后使用记事本新建一个文件，将文件名修改为“static.youku.com”（即扩展名为“.com”），并把属性改为只读，这样Flash就无法正常读写SharedObjects，从而无法判断是否屏蔽广告了。

（如果用的是Google Chrome浏览器，则还要改

`C:\Users\Administrator\AppData\Local\Google\Chrome\UserData\Default\PepperData\Shockwave Flash\WritableRoot\#SharedObjects\[随机数]\static.youku.com`

linux系统下是`/home/[username]/.config/google-chrome/Default/Pepper Data/Shockwave Flash/WritableRoot/#SharedObjects/[随机数]/static.youku.com`


<!--more-->

## 屏蔽广告是与非

用户屏蔽网站广告“自古有之”，从Firefox的Adblock Plus，到修改Hosts，再到360安全卫士类似软件的附加功能，可谓手段越来越恨，效果越来越好。于是乎网站就不干了，原本就是免费为用户提供内容，靠着广告收入回收成本（盈利与否另说）。用户都做了广告屏蔽，这样一来仅有的广告收入也减少了。更不用说视频网站昂贵的带宽成本……

当然用户也有用户的“道理”，诸如广告质量低劣，广告音量大且不可静音，广告时间长且不可跳过（30秒广告15秒视频不是没有见过）……

这样说来似乎是不可调和的矛盾了，折中一下吧，广告拍得让广大网民喜闻乐见一些，再学学YouTube的广告投递方式，相信用户还是可以买账的。再说网站没了收入，用户不就没得看了吗？这样的后果相信谁也不会希望看到的。

原文地址：http://joys.name/2011/09/block-youku-ad.html

附加：自从youku与土豆合并之后立，当观众在准备观看每一个视频前，片头的广告将会略有增加，变为youku的45秒，加上tudou的30秒.HostsX

```xml
#去优酷广告 

0.0.0.0 stat.youku.com 

0.0.0.0 static.lstat.youku.com 

0.0.0.0 valb.atm.youku.com 

0.0.0.0 valc.atm.youku.com 

0.0.0.0 valf.atm.youku.com 

0.0.0.0 valo.atm.youku.com 

0.0.0.0 valp.atm.youku.com 

0.0.0.0 vid.atm.youku.com 

0.0.0.0 walp.atm.youku.com

#去土豆网广告 

127.0.0.1 adextensioncontrol.tudou.com 

127.0.0.1 iwstat.tudou.com 

127.0.0.1 nstat.tudou.com 

127.0.0.1 stats.tudou.com 

127.0.0.1 *.p2v.tudou.com* 

127.0.0.1 at-img1.tdimg.com 

127.0.0.1 at-img2.tdimg.com 

127.0.0.1 at-img3.tdimg.com 

127.0.0.1 adplay.tudou.com 

127.0.0.1 adcontrol.tudou.com 

127.0.0.1 stat.tudou.com

#去酷6网广告 

127.0.0.1 1.allyes.com.cn 

127.0.0.1 analytics.ku6.com 

127.0.0.1 gug.ku6cdn.com 

127.0.0.1 ku6.allyes.com 

127.0.0.1 ku6afp.allyes.com 

127.0.0.1 pq.stat.ku6.com 

127.0.0.1 st.vq.ku6.cn 

127.0.0.1 stat0.888.ku6.com 

127.0.0.1 stat1.888.ku6.com 

127.0.0.1 stat2.888.ku6.com 

127.0.0.1 stat3.888.ku6.com 

127.0.0.1 static.ku6.com 

127.0.0.1 v0.stat.ku6.com 

127.0.0.1 v1.stat.ku6.com 

127.0.0.1 v2.stat.ku6.com 

127.0.0.1 v3.stat.ku6.com

#去奇艺广告 

127.0.0.1 afp.qiyi.com 

127.0.0.1 focusbaiduafp.allyes.com

#去CNTV 

127.0.0.1 a.cctv.com

127.0.0.1 ad.cctv.com 

127.0.0.1 d.cntv.cn 

127.0.0.1 adguanggao.eee114.com 

127.0.0.1 cctv.adsunion.com 

#新浪视频 

127.0.0.1 dcads.sina.com.cn

#pptv 

127.0.0.1 pp2.pptv.com

#乐视 

127.0.0.1 pro.letv.com 

#搜狐高清 

127.0.0.1 images.sohu.com 

#HostsX 国内站点广告/视频类网站 

#CNTV 

127.0.0.1 a.cctv.com 

127.0.0.1 a.cntv.cn 

127.0.0.1 ad.cctv.com 

127.0.0.1 d.cntv.cn 

127.0.0.1 adguanggao.eee114.com 

127.0.0.1 cctv.adsunion.com 

#我乐网 

127.0.0.1 acs.56.com 

127.0.0.1 acs.agent.56.com 

127.0.0.1 acs.agent.v-56.com 

127.0.0.1 bill.agent.56.com 

127.0.0.1 bill.agent.v-56.com 

127.0.0.1 stat.56.com 

127.0.0.1 stat2.corp.56.com 

127.0.0.1 union.56.com 

127.0.0.1 uvimage.56.com 

127.0.0.1 v16.56.com 

#6间房 

127.0.0.1 pole.6rooms.com 

127.0.0.1 shrek.6.cn 

127.0.0.1 simba.6.cn 

127.0.0.1 union.6.cn 

#激动网 

127.0.0.1 86file.megajoy.com 

127.0.0.1 86get.joy.cn 

127.0.0.1 86log.joy.cn 

#天线视频 

127.0.0.1 casting.openv.com 

127.0.0.1 m.openv.tv 

127.0.0.1 uniclick.openv.com 

#迅雷看看屏蔽： 

127.0.0.1 mcfg.sandai.net 

127.0.0.1 biz5.sandai.net 

127.0.0.1 server1.adpolestar.net 

127.0.0.1 advstat.xunlei.com 

127.0.0.1 mpv.sandai.net

```