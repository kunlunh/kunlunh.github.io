---
author: HKL
categories:
- 默认分类
cid: 50
date: "2018-06-13T20:08:00Z"
slug: asterisk-trunk-sip-iax2
status: publish
tags:
- Operating
- Linux
- Firefox
title: asterisk服务器的sip、iax2中继对接
updated: 2019/01/29 16:26:10
---


一、sip中继
1.Asterisk 互連～ SIP Provider 模式
http://www.osslab.org.tw/VoIP/IP_PBX/ 軟體式_IP_PBX/Asterisk/Tips/Asterisk_互連～_SIP_Provider_模式

什麼是 SIP Provider 模式？ 當 Asterisk(provider) 想成為一個類似 SIP Provider 的應用，讓另一台 Asterisk(end) 可以註冊，並且可以透過 Asterisk(provider) 的 Trunk 撥出以及也可以撥到 Asterisk(provider) 所有分機，但此模式的 Asterisk(end) 只是扮演 user，所以它的所有分機是與 Asterisk(provider) 是隔絕的，而且可以使用 Inbount Route 來控制 Asterisk(provider) 的來電。

簡單來說，要使兩台 Asterisk 互連，如果其目的是僅限其中一方的資源被對方使用，應該使用 SIP Provider 模式，反之，若互連的目的是使雙方的資源都可以彼此互用，就像總公司與分公司關係，兩地的分機群必須完全互通，就好像是同一個主機一般，這類的應用請參考另一篇教學。

IP-PBX Asterisk 使用 IAX 互連設定~總整理 


<!--more-->


系統環境說明

Asterisk(provider)：這台將作為類似 SIP Provider 應用，對方可以使用這裡的 Trunk 及與這裡的所有分機互通。 分機號：1XX 主機 IP: 192.168.1.1

Asterisk(end)：這台將作為類似 SIP User 應用，這裡的所有分機可以撥到對方的所有分機，但對方分機不可直接撥入，所有對方來電都可以由這裡的 Inbound Route 來作控制。 分機號：2XX 主機 IP：192.168.1.2

設定開始

所有步驟以 Elastix 的 FreePBX 管理介面操作為例。 在 Asterisk(provider)

**新增一個 SIP 分機 ** PBX -> PBX Configuration -> Extensions 分機號：199 註冊密碼：199pass

PS.這裡的步驟與一般分機設置相同 在 Asterisk(end)

新增一個 SIP Trunk 註冊於 Asterisk(provider)

```ini
Trunk Name: ast_provider 
PEER Details: 
username=199 
type=peer 
secret=199pass 
insecure=very 
host=192.168.1.1 
fromuser=199 
qualify=yes

Incoming Settings 
USER Context: 空白 
USER Details: 空白

Register String: 
199:199pass@192.168.1.1/ast_provider_reg
```

PS.最後面為甚麼不是 SIP number 而是改用字串呢？這是因為若以 SIP number 199 來作識別，可能會與本地的其他分機的編碼規則造成衝突，所以改用字串可以避免爾後遇到路由的問題。

**新增 Inbound Route**

Description: 自行定義 DID Number: ast_provider_reg <– 名稱必須與 register string 最後面的字串相同。 Set Destinaion: 這裡可以指定任一分機、分機群組、IVR等等。

PS. 儲存設定時，系統可能會提示 DID number 不可輸入英文名的警告，請按確定即可。

**新增 Outbound Route**

```ini
Route Name: 自行定義 
Dial Patterns: 
012|.
```

`Trunk Sequence: SIP/ast_provider`

PS. 本例使用 Prefix code 012，只要撥到這個 Trunk 的號碼，除了對方的號碼外，撥號前還需要先加上 012，例如： 當撥到對方(SIP_A)分機 101 時，在 SIP_B 要撥 012101 當撥到對方(SIP_A)外線時 861234567，在 SIP_B 要撥 012861234567

2.多台asterisk使用SIP对接

http://www.nbao.net/post/2010/05/11/e5a49ae58fb0asteriske4bdbfe794a8SIPe5afb9e68ea5.aspx 

当用户数量上去,单凭一台asterisk是很能支持庞大的用户群体,所以要根据用户量来部署多台asterisk来应付实际情况的需求.但部署多台asterisk所带来的一个问题就是A服务器的用户如果Call B服务器的用户呢?其实asterisk的设计者早已帮我们解决问题,以下是通过SIP把两台asterisk对接起来(不过官方推荐asterisk的对接用AIX).

分别在192.168.1.21 和192.168.1.22两台服务器上装上asterisk,然后配置各自的用户，TRUNK和转发规则。

配置192.168.1.21

打开`/etc/asterisk/sip.conf`

有[general]组下添加注册到22的命令

`register=>AST22:123456@192.168.1.22`

然后在文件尾添加相关组信息

```ini
[AST21]
type=friend
secret=123456
host=dynamic
username=AST21
disallow=all
allow=ulaw;alaw
context=FROMSIP

[22TRUNK]
type=friend
username=AST22
secret=123456
host=192.168.1.22
dtmfmode=rfc2833
context=FROMSIP
fromuser=AST22
insecure=very
```

打开`/etc/asterisk/extensions.conf` 添加下面内容

```ini
[FROMSIP]
Exten => _90.,1,dila(sip/91${exten:2}@22TRUNK,40,m(default))
```

拔打90开头的号码，把91代替90后转发192.168.1.22,拔打等待40秒，等待的时候播放default这个采铃。

```ini
Exten => _91.,1,dial(sip/${exten:2},40,m(default))
```

当接收到91开头的号码，把91后面的号码进行内部呼叫。

配置192.168.1.22

打开`/etc/asterisk/sip.conf`

有`[general]`组下添加注册到22的命令

```ini
register=>AST21:123456@192.168.1.21
```

然后在文件尾添加相关组信息

```ini
[AST22]
type=friend
secret=123456
host=dynamic
username=AST22
disallow=all
allow=ulaw;alaw
context=FROMSIP

[21TRUNK]
type=friend
username=AST21
secret=bsmofeng
host=192.168.1.21
fromuser=AST21
dtmfmode=rfc2833
context=FROMSIP
insecure=very
```

打开`/etc/asterisk/extensions.conf` 添加下面内容

```ini
[FROMSIP]
Exten => _90.,1,dila(sip/91${exten:2}@21TRUNK,40,m(default))
```

拔打90开头的号码，把91代替90后转发192.168.1.21,拔打等待40秒，等待的时候播放default这个采铃。

```ini
Exten => _91.,1,dial(sip/${exten:2},40,m(default))
```

当接收到91开头的号码，把91后面的号码进行内部呼叫。

这样就配置好了两台asterisk的sip对接，不过当用户数量庞大和分布在不同地区显然2台asterisk不足以应付的。在N台asterisk下通过手动配置conf文件来实现对接是不可能的，因为用户会根据不同情况可能登陆不同的asterisk里，在这情况exten是无法固下来；这个时候就可能采asterisk的AMI和AGI来动态处理，通过AMI来获取号码登陆的asterisk服务器地址,AGI在根据号码所在asterisk做一个动态的TRUNK拔打就行。

作为一个语音较交换服务器asterisk的确算是一个好的产品，他除了开源外，还提供AMI，AGI等接口；使其他语言平台通过这些接口来扩展自己的业务需求。

2.iax2中继
(1)asterisk iax互联 
http://wenku.baidu.com/view/ed06e74ffe4733687e21aa1c.html 或

IP-PBX Asterisk 使用 IAX 互連設定~總整理 
http://docs.google.com/Doc?id=dqzwkb4_32gqbvncgr

(2)IAX 设置详细（zt）
http://www.cn-cti.com/681.html

linux下面配置IAX（ZT）
http://www.cn-cti.com/679.html

(3)连接两台asterisk服务器 
http://www.dinghong.org/2008/07/10

有两台asterisk服务器，需要可以拨打注册在对方服务器上的分机号。

假设有A ，B两台服务器，A上面分机号都以3开头，如3000；B上面分机号都以8开头，如8000。

在A上新建iax Trunk，命名`"InterOffice"`，配置如下：

`peer detail`项

```ini
host=B的ip地址 
Qualify=yes 
type=friend 
```

另外定一个拨号规则，这里是`”80xx”`

在A上建Outbound Routes，命名`”InterOffice”`，配置如下： 拨号规则`”80xx”`，trunk选`"InterOffice"`

在B上建iax Trunk，命名`"InterOffice"`，配置如下：

```ini
"peer detail"项 
host=A的ip地址 
Qualify=yes 
type=friend 
```

另外定一个拨号规则，如果需要的话，这里`”30xx”`

在B上建Outbound Routes，命名`"InterOffice"`，配置如下： 拨号规则`”30xx”`，trunk选`"InterOffice"`

测试：从A拨打8000（注册在B上），能接通；从B拨打3000（注册在A上），能接通。

在一台asterisk服务器上拨号，电话从另一台打出 
http://www.dinghong.org/2008/07/12

假设有A ，B两台服务器，要实现在服务器A上拨打外线电话，电话从B服务器打出。

首先两台asterisk服务器要互通，在前面”连接两台asterisk服务器”文章里已经有说过怎么配置。

修改A服务器上连通到B服务器的Outbound Routes ，拨号规则改成`"4|."`

（注意不要和别的Outbound Routes里的拨号规则一样），现在依然可以拨打4+分机号打到B上面的分机上。

修改B服务器上trunk配置，在"PEER Details"里加一条`"context=from-internal"`，

假设B上面有条`”Outbound Routes”`拨号规则为`"2|."`拨打外线号码，

那么现在在A服务器上就可以通过拨`"42+电话号码"`从B服务器打电话出去。