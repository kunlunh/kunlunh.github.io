---
author: HKL
categories:
- 默认分类
date: "2019-12-24T23:25:00Z"
slug: openwrt-asterisk-dongle-gsm-to-sip
status: publish
tags:
- Networking
- Operating
title: OpenWRT通过3G Modem加asterisk将GSM通话转为SIP
---

本文主要实现OpenWRT系统通过Huawei 3G Modem加asterisk套件将GSM通话转为SIP通话


安装openwrt下的asterisk16套件

```bash
opkg update
opkg install asterisk16-app-system asterisk16-chan-dongle asterisk16-pjsip asterisk16-codec-ulaw asterisk16-codec-alaw asterisk16-res-rtp-asterisk asterisk16-bridge-simple
```

调整PJSIP作为默认服务，并且新增几个PJSIP账户，用以测试内线通

<!--more-->

`/etc/asterisk/pjsip.conf`

```bash
[transport-udp]                                                        
type=transport   
protocol=udp    ;udp,tcp,tls,ws,wss                                        
bind=0.0.0.0

[6003]                                                                          
type=endpoint                                                                 
transport=transport-udp                                                      
context=from-internal                                 
disallow=all                                
allow=ulaw                                                              
auth=6003-auth                                                                
aors=6003                                                                    
                                                                               
[6003-auth]                                                                             
type=auth                                                                       
auth_type=userpass                                                      
password=6003                                                                     
username=6003                                                                 
                                                                            
[6003]                                                                        
type=aor                                                                       
max_contacts=1



[6004]                                                                 
type=endpoint                                                                 
transport=transport-udp                                                        
context=from-internal                                                              
disallow=all                                                                    
allow=ulaw                                                              
auth=6004-auth                                                                     
aors=6004                                                                     
                                                                            
[6004-auth]                                                                   
type=auth                                                                      
auth_type=userpass                                                              
password=6004                                                                   
username=6004                                                                  
                                                                               
[6004]                                                                      
type=aor                                                                              
max_contacts=1 

```

新增分机拨打模板，

`/etc/asterisk/extension.conf`

```bash
[from-internal]                                                               
exten => _Z.,1,Dial(PJSIP/${EXTEN},60,Trg)                                        
same => n,Hangup() 
```

用asterisk查看分机状态，拨打过程中`pjsip show endpoints`中显示的状态会从`Not in use`转换为`In use`

```bash
asterisk -rvvvv
OpenWrt*CLI> pjsip show contacts

  Contact:  <Aor/ContactUri..............................> <Hash....> <Status> <RTT(ms)..>
==========================================================================================

  Contact:  6001/sip:6001@192.168.234.127:53117;transport= 378e2db08b NonQual         nan
  Contact:  6001/sip:6001@192.168.234.127:53117;transport= 378e2db08b NonQual         nan
  Contact:  6003/sip:6003@192.168.234.158:49989;transport= 2ec100f865 NonQual         nan
  Contact:  6003/sip:6003@192.168.234.158:49989;transport= 2ec100f865 NonQual         nan
  Contact:  6004/sip:6004@192.168.104.11:5060              586381001a NonQual         nan
  Contact:  6004/sip:6004@192.168.104.11:5060              586381001a NonQual         nan

Objects found: 6

    -- PJSIP/6001-00000005 is ringing
    -- PJSIP/6001-00000005 is ringing
       > 0x2262b00 -- Strict RTP learning after remote address set to: 192.168.234.127:52518
    -- PJSIP/6001-00000005 answered PJSIP/6004-00000004
       > 0x2270c60 -- Strict RTP learning after remote address set to: 192.168.104.11:11866
    -- Channel PJSIP/6001-00000005 joined 'simple_bridge' basic-bridge <ee120657-8627-4868-b677-cb0d896a2b5a>
    -- Channel PJSIP/6004-00000004 joined 'simple_bridge' basic-bridge <ee120657-8627-4868-b677-cb0d896a2b5a>
       > 0x2262b00 -- Strict RTP switching to RTP target address 192.168.234.127:52518 as source
       > 0x2270c60 -- Strict RTP switching to RTP target address 192.168.104.11:11866 as source
OpenWrt*CLI> pjsip show endpoints

 Endpoint:  <Endpoint/CID.....................................>  <State.....>  <Channels.>
    I/OAuth:  <AuthId/UserName...........................................................>
        Aor:  <Aor............................................>  <MaxContact>
      Contact:  <Aor/ContactUri..........................> <Hash....> <Status> <RTT(ms)..>
  Transport:  <TransportId........>  <Type>  <cos>  <tos>  <BindAddress..................>
   Identify:  <Identify/Endpoint.........................................................>
        Match:  <criteria.........................>
    Channel:  <ChannelId......................................>  <State.....>  <Time.....>
        Exten: <DialedExten...........>  CLCID: <ConnectedLineCID.......>
==========================================================================================

 Endpoint:  6001                                                 In use        1 of inf
     InAuth:  6001-auth/6001
        Aor:  6001                                               1
      Contact:  6001/sip:6001@192.168.234.127:53117;transp 378e2db08b NonQual         nan
  Transport:  transport-udp             udp      0      0  0.0.0.0:5060
    Channel: PJSIP/6001-00000005/AppDial                         Up            00:00:04   
        Exten:                           CLCID: "6004" <6004>

 Endpoint:  6002                                                 Unavailable   0 of inf
     InAuth:  6002-auth/6002
        Aor:  6002                                               1
  Transport:  transport-udp             udp      0      0  0.0.0.0:5060

 Endpoint:  6003                                                 Not in use    0 of inf
     InAuth:  6003-auth/6003
        Aor:  6003                                               1
      Contact:  6003/sip:6003@192.168.234.158:49989;transp 2ec100f865 NonQual         nan
  Transport:  transport-udp             udp      0      0  0.0.0.0:5060

 Endpoint:  6004                                                 In use        1 of inf
     InAuth:  6004-auth/6004
        Aor:  6004                                               1
      Contact:  6004/sip:6004@192.168.104.11:5060          586381001a NonQual         nan
  Transport:  transport-udp             udp      0      0  0.0.0.0:5060
    Channel: PJSIP/6004-00000004/Dial                            Up            00:00:04   
        Exten: 6001                      CLCID: "" <6001>


Objects found: 4

       > 0x2270c60 -- Strict RTP learning complete - Locking on source address 192.168.104.11:11866
       > 0x2262b00 -- Strict RTP learning complete - Locking on source address 192.168.234.127:52518


```

有条件的情况下建议可以考虑使用IAX分机替代SIP分机，这样只需要NAT打通一个UDP端口就能通话，而不用像SIP那样要考虑ALG,ICE,STUN等方案

下面是新增一个IAX分机的用例

```bash
opkg update
opkg install asterisk16-chan-iax2
```

`/etc/asterisk/iax.conf`

```bash
[general]
bindport=4569
bindaddr=0.0.0.0
iaxcompat=yes
nochecksums=yes
disallow=all
allow=ulaw

[6010]
type=friend
username=6010
secret=6010
context=from-internal
host=dynamic
callerid=6010<6010>
```

`/etc/asterisk/extensions.conf`

```bash
[from-internal]
exten => 6010,1,Dial(IAX2/6010,60,Trg)
```

由于3G modem还没到货，所以目前先更新到这里，等到货后继续配置。

12月27号收货，继续更，用到的型号是Huawei K3765.

![K3765 3G modem 1][1]

![K3765 3G modem 2][2]
K3765 3G modem


在openwrt下配置dongle设备,请结合实际数据配置

`/etc/asterisk/dongle.conf`

```bash
[general]
interval=20
[defaults]
context=dongle-in
group=0
exten=+862022221234
[dongle0]
imei=123451234512345
```

通过asterisk控制台查一下设备状态,

```bash
asterisk -rvvvv
RiWifi*CLI> dongle show devices
ID           Group State      RSSI Mode Submode Provider Name  Model      Firmware          IMEI             IMSI             Number        
dongle0      0     Free       16   3    3       FFFFFFFFFFFFFF K3765      11.126.03.04.521  123451234512345  123451234512345  Unknown 
```

接下来新增呼出、呼入的分机配置

`/etc/asterisk/extensions.conf`

```bash
[from-internal]
exten => _1.,1,Dial(Dongle/g0/${EXTEN:1}) ;呼出设置，结合实际，我这边是加了"1"这个前缀，例如我的SIP分机要拨打10011,那么拨号就是110011
[dongle-in]
exten => +862022221234,1,Dial(IAX2/6010,60,Trg) ;呼入设置，我这边就是配置成所有呼叫直接转到IAX-6010分机，复杂点的可以做IVR，号码本，不过只有一路的电话就不需要搞这么复杂了。
```


最后，拨号测试

呼入测试

![呼入测试1][3]

![呼入测试2][4]

呼出测试

![呼出测试1][5]

![呼出测试2][6]

  [1]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/rr971rwayw.png
  [2]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/o8navu8t3a.jpeg
  [3]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/ppvuess1u8.png
  [4]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/bu16kf7yx3.jpeg
  [5]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/r6wjvf9s6k.png
  [6]: https://cdn.jsdelivr.net/gh/hiplon/blog-photo/2019/11/jek9b1pyhn.jpeg